<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Setoran Uang
      </h3>
    </div>
    <div class="card-body">

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <div class="mt-2 accordion" role="tablist">
        <div class="card mb-1" v-for="u in setorUang" role="tab">
          <div class="card-header" style="margin:0; padding:0;" v-if="u.setor_uang.length > 0">
            <b-button variant="info" class="text-left" block style="border-radius:6px"
            v-b-toggle="'acc-' + u.id_users">
              <b-button variant="danger" size="sm" @click="drop(u.id_users)">
                <span class="fa fa-trash"></span>
              </b-button>
              {{ u.nama }}
              <span class="text-dark">
                [{{ "Rp " + formatNumber(getTotal(u.setor_uang)) }}]
              </span>
            </b-button>
          </div>

          <b-collapse :id="'acc-' + u.id_users" accordion="my-accordion" role="tabpanel"
          v-if="u.setor_uang.length > 0">
            <div class="card-body">
              <ul class="list-group">
                <li class="list-group-item">
                   <b-row>
                     <b-col class="mb-1" cols="6" v-for="(s,i) in u.setor_uang">
                       {{ "Bill " + (i+1) }}: {{ "Rp " + formatNumber(s.nominal) }}
                     </b-col>
                   </b-row>
                </li>
              </ul>
            </div>
          </b-collpase>
        </div>
      </div>
      <b-button class="mt-2" block variant="success" @click="addSetoran()"
      v-b-modal.modal_setoran>
        Tambah Setoran Uang
      </b-button>
  </div>

  <b-modal id="modal_setoran" title="Tambah Data Setoran Uang"
    header-bg-variant="success" size="lg"
    border-variant="info" hide-footer>
    <form v-on:submit.prevent="SetoranUang">
      <b-container fluid>
        <b-row class="mb-1">
          <b-col cols="4">Customer</b-col>
          <b-col>
            <v-select :options="customers" @input="bindCustomer()"
            :reduce="nama => nama.id_users"
            label="nama" v-model="selectedUser.id_users">
            <template #search="{attributes, events}">
              <input
                class="vs__search"
                :required="!selectedUser.id_users"
                v-bind="attributes"
                v-on="events"
              />
            </template>
            </v-select>
          </b-col>
        </b-row>

        Tanggungan Pembayaran
        <b-row class="mb-1">
          <b-col>
            <b-form-checkbox-group :options="selectedUser.bill" value-field="id_orders"
             text-field="label" v-model="selectedUser.setor_uang">
           </b-form-checkbox-group>
          </b-col>
        </b-row>

        <b-row v-if="selectedUser.setor_uang.length > 0" class="mb-2">
          <b-col class="text-blue">
            {{ "Total Setor: Rp " + formatNumber(getTotalSetor()) }}
          </b-col>
        </b-row>
        <button type="submit" class="btn btn-block btn-dark">
          Simpan Setoran Uang
        </button>
      </b-container>
    </form>
  </b-modal>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        customers: [],
        pack: [],
        setorUang: [],
        id_sopir: null,
        message : "",
        key: "",
        selectedUser: {
          id_users: "",
          setor_uang: [],
          bill: [],
        }
      }
    },

    methods: {
      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          total += Number(item.nominal)
        });
        return total;
      },

      getTotalSetor : function(){
        let total = 0;
        this.selectedUser.setor_uang.forEach((item, i) => {
          let it = this.selectedUser.bill.find(b => b.id_orders === item);
          total += Number(it.nominal)
        });
        return total;
      },

      bindCustomer : function(){
        let id = this.selectedUser.id_users;
        let item = this.customers.find(it => it.id_users === id);
        item.bill.forEach((it, i) => {
          this.selectedUser.bill.push({
            label: "Bill " + (Number(i) + 1) + ": Rp " + formatNumber(it.nominal),
            id_orders: it.id_orders,
            id_bill: it.id_bill,
            nominal: it.nominal
          })
        });
      },

      addSetoran : function(){
        this.selectedUser.id_users = "";
        this.selectedUser.setor_uang = [];
        this.selectedUser.bill = [];
      },

      getSetoranUang : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/setor-uang/" + this.users.id_users, conf)
        .then(response => {
          this.setorUang = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      getTanggunganUsers : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/customer-tagihan", conf)
        .then(response => {
          this.customers = response.data.filter(it => it.bill.length > 0);
        })
        .catch(error => {
          console.log(error);
        })
      },

      SetoranUang : function(){
        let setorUang = this.selectedUser.setor_uang;
        if (setorUang.length > 0) {
          if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
            this.$bvToast.show("loading");
            this.$bvModal.hide("modal_setoran");
            let conf = { headers: { "Api-Token" : this.key} };
            let form = new FormData();
            form.append("id_users", this.users.id_users);
            form.append("id_pembeli", this.selectedUser.id_users);
            form.append("setor_uang", JSON.stringify(setorUang));
            axios.post(base_url + "/setor-uang", form, conf)
            .then(response => {
              this.$bvToast.hide("loading");
              this.message = response.data.message;
              this.$bvToast.show("message");
              this.getTanggunganUsers();
              this.getSetoranUang();
            })
            .catch(error => {
              console.log(error);
            })
          }
        }else{
          alert("Tidak ada tagihan yang dibayarkan")
        }
      },

      drop : function(id_pembeli){
        if (confirm("Apakah Anda yakin ingin menghapus data ini?")) {
          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          axios.delete(base_url + "/setor-uang/" + this.users.id_users + "/" + id_pembeli, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.getSetoranUang();
            this.getTanggunganUsers();
          })
          .catch(error => {
            console.log(error);
          })
        }
      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/driver/check", form)
        .then(response => {
          if (response.data.auth == false) {
            window.location = "login.html";
          } else{
            this.key = token;
            this.users = response.data.driver;
            this.getSetoranUang();
            this.getTanggunganUsers();
          }
        })
        .catch(error => {
          console.log(error);
        });
      },

    },

    mounted(){
      this.initUser();
    }
  }
</script>
