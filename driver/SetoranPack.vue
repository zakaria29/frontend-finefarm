<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Setoran Pack
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
        <div class="card mb-1" v-for="u in kembaliPack" role="tab">
          <div class="card-header" style="margin:0; padding:0;" v-if="u.kembali_pack.length > 0">
            <b-button variant="info" class="text-left" block style="border-radius:6px"
            v-b-toggle="'acc-' + u.id_users">
              {{ u.nama }}
            </b-button>
          </div>

          <b-collapse :id="'acc-' + u.id_users" accordion="my-accordion" role="tabpanel"
          v-if="u.kembali_pack.length > 0">
            <div class="card-body">
              <ul class="list-group">
                <li class="list-group-item">
                   <b-row>
                     <b-col class="mb-1" cols="4" v-for="k in u.kembali_pack">
                       <b-button variant="danger" @click="drop(k.id_pembeli,k.id_pack)"
                       size="sm">
                         <span class="fa fa-trash"></span>
                       </b-button>
                       {{ k.pack.nama_pack }}: {{ k.jumlah }}
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
        Tambah Setoran Pack
      </b-button>
  </div>

  <b-modal id="modal_setoran" title="Tambah Data Setoran Pack"
    header-bg-variant="success" size="lg"
    border-variant="info" hide-footer>
    <form v-on:submit.prevent="SetoranPack">
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

        Tanggungan Pack
        <b-row class="mb-1">
          <b-col>
            <ul class="list-group">
              <li class="list-group-item">
                <b-row>
                  <b-col>
                    <strong> <small>Nama Pack</small> </strong>
                  </b-col>
                  <b-col class="text-center">
                    <strong> <small>Jml. Tanggungan</small> </strong>
                  </b-col>
                  <b-col class="text-center">
                    <strong> <small>Jml.  Kembali</small> </strong>
                  </b-col>
                </b-row>
              </li>
              <li class="list-group-item" v-for="p in selectedUser.tanggungan_pack">
                <b-row v-if="p.max > 0">
                  <b-col>
                    <small>{{ p.nama_pack }}</small>
                  </b-col>
                  <b-col class="text-center">
                    <small>{{ p.max }}</small>
                  </b-col>
                  <b-col>
                    <b-form-input v-model="p.jumlah" :max="p.max" type="number" size="sm">
                    </b-form-input>
                  </b-col>
                </b-row>
              </li>
            </ul>
          </b-col>
        </b-row>
        <button type="submit" class="btn btn-block btn-dark">
          Simpan Setoran Pack
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
        kembaliPack: [],
        id_sopir: null,
        message : "",
        key: "",
        selectedUser: {
          id_users: "",
          tanggungan_pack: []
        }
      }
    },

    methods: {
      bindCustomer : function(){
        let id = this.selectedUser.id_users;
        this.selectedUser.tanggungan_pack = [];
        let item = this.customers.find(it => it.id_users === id);
        item.tanggungan_pack.forEach((it, i) => {
          this.selectedUser.tanggungan_pack.push({
            id_pack: it.id_pack,
            nama_pack: it.pack.nama_pack,
            jumlah: 0,
            max: it.jumlah,
            id_users: this.users.id_users,
            id_pembeli: id
          })
        });
      },

      addSetoran : function(){
        this.selectedUser.id_users = "";
        this.selectedUser.tanggungan_pack = [];
      },

      getSetoranPack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/kembali-pack/" + this.users.id_users, conf)
        .then(response => {
          this.kembaliPack = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      getTanggunganUsers : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/customer-pack", conf)
        .then(response => {
          this.customers = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      SetoranPack : function(){
        let setorPack = this.selectedUser.tanggungan_pack.filter(it => it.jumlah > 0);
        if (setorPack.length > 0) {
          if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
            this.$bvToast.show("loading");
            this.$bvModal.hide("modal_setoran");
            let conf = { headers: { "Api-Token" : this.key} };
            let form = new FormData();
            form.append("id_users", this.users.id_users);
            form.append("id_pembeli", this.selectedUser.id_users);
            form.append("kembali_pack", JSON.stringify(setorPack));
            axios.post(base_url + "/kembali-pack", form, conf)
            .then(response => {
              this.$bvToast.hide("loading");
              this.message = response.data.message;
              this.$bvToast.show("message");
              this.getSetoranPack();
            })
            .catch(error => {
              console.log(error);
            })
          }
        }else{
          alert("Tidak ada pack yang dikembalikan")
        }
      },

      drop : function(id_pembeli,id_pack){
        if (confirm("Apakah Anda yakin ingin menghapus data ini?")) {
          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          axios.delete(base_url + "/kembali-pack/" + id_pembeli + "/" + id_pack, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.getSetoranPack();
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
            this.getSetoranPack();
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
