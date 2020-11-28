<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-users"></span>
        Data Customer
      </h3>
    </div>
    <div class="card-body">
      <form v-on:submit.prevent="find">
         <b-form-input v-model="search" class="mb-1" placeholder="Pencarian...">
         </b-form-input>
      </form>
      <b-table striped hover :items="user" :fields="fields">
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item)"
          v-b-modal.modal_user>
            <span class="fa fa-edit"></span>
          </b-button>
          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_users)">
            <span class="fa fa-trash"></span>
          </b-button>
          <b-button class="btn btn-sm btn-warning" @click="Lock(data.item)"
          v-b-modal.modal_lock>
            <span class="fa fa-unlock-alt"></span> Lock Barang
          </b-button>
        </template>
        <template v-slot:cell(nama)="data">
          <img :src="storage_path + data.item.image"
          class="avatar avatar-md rounded-circle">
          <span class="ml-3">{{ data.item.nama }}</span>
        </template>
      </b-table>

      <b-pagination
      size="md"
      class="mt-3"
      :total-rows="totalRow"
      v-model="currentPage"
      :per-page="perPage"
      align="center"
      v-on:input="find">
      </b-pagination>
    </div>
    <div class="card-footer">
      <b-button class="btn btn-sm btn-success" v-b-modal.modal_user
      @click="Add">
      <span class="fa fa-plus"></span> Tambah Data
      </b-button>
    </div>

    <!-- toast -->
    <!-- toast untuk tampilan message box -->
    <b-toast id="message" title="Message" solid>
      <strong class="text-success">{{ message }}</strong>
    </b-toast>

    <b-toast id="loading" title="Processing Data" solid no-auto-hide>
      <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
      <span class="fa fa-spinner fa-spin"></span>
      <strong class="text-success">Loading...</strong>
    </b-toast>
    <!-- Modal Component -->
    <b-modal id="modal_lock" title="Lock Barang dan Pack"
    header-bg-variant="info" size="lg"
    border-variant="info" hide-footer>
      <b-container fluid>
        <b-row class="mb-2">
          <b-col cols="3">Customer</b-col>
          <b-col>: {{ users.nama }}</b-col>
        </b-row>
        <form v-on:submit.prevent="SaveLock">
          <b-row>
            <b-col>
              Lock Pack, Barang, dan Harga
              <ul class="list-group">
                <li class="list-group-item">
                  <b-row class="mb-1">
                    <b-col cols="3">
                      <strong> <small>Barang</small> </strong>
                    </b-col>
                    <b-col cols="3">
                      <strong> <small>Pack</small> </strong>
                    </b-col>
                    <b-col cols="4">
                      <strong> <small>Harga</small> </strong>
                    </b-col>
                    <b-col cols="2">
                      <b-button size="sm" variant="success" @click="addLock">
                        <span class="fa fa-plus"></span>
                      </b-button>
                    </b-col>
                  </b-row>
                </li>

                <li class="list-group-item" v-for="(s,i) in selectedLockPackBarang">
                  <b-row class="mb-1">
                    <b-col cols="3">
                      <b-form-select size="sm" v-model="s.id_barang" @change="bindBarang(i)"
                      required>
                        <option v-for="b in barang" :value="b.id_barang">
                          {{ b.nama_barang }}
                        </option>
                      </b-form-select>
                    </b-col>
                    <b-col cols="3">
                      <b-form-select size="sm" v-model="s.id_pack" required>
                        <option v-for="p in s.pack" :value="p.id_pack">
                          {{ p.nama_pack }}
                        </option>
                      </b-form-select>
                    </b-col>
                    <b-col cols="4">
                      <b-form-input v-model="s.harga" type="number" size="sm" required>
                      </b-form-input>
                    </b-col>
                    <b-col cols="2">
                      <b-button size="sm" variant="danger" @click="dropLock(i)">
                        <span class="fa fa-trash"></span>
                      </b-button>
                    </b-col>
                  </b-row>
                </li>
              </ul>

              <b-button block type="submit" variant="primary">
                Simpan
              </b-button>
            </b-col>
          </b-row>
        </form>
      </b-container>
    </b-modal>
 <b-modal
     id="modal_user"
     title="Form Customer"
     header-bg-variant="info"
     border-variant="info" hide-footer>

     <b-container fluid>
       <form v-on:submit.prevent="Save">
         Nama
         <b-form-input v-model="users.nama" class="mb-1" required>
         </b-form-input>
         Alamat
         <b-form-input v-model="users.alamat" class="mb-1" required>
         </b-form-input>
         Contact
         <b-form-input v-model="users.contact" class="mb-1" required>
         </b-form-input>
         Email
         <b-form-input v-model="users.email" class="mb-1" required>
         </b-form-input>
         Username
         <b-form-input v-model="users.username" class="mb-1" required>
         </b-form-input>
         Password
         <b-form-input type="password" v-model="users.password" class="mb-1" required>
         </b-form-input>
         Image
         <b-form-file id="image" v-model="users.image" class="mb-1"
         :required="users.require">
         </b-form-file>
         Status
         <b-form-select v-model="users.status" class="mb-1">
           <option value="0">Tidak Aktif</option>
           <option value="1">Aktif</option>
         </b-form-select>
         Margin
         <b-form-input type="number" v-model="users.margin" class="mb-1" required>
         </b-form-input>
         Nama Instansi
         <b-form-input v-model="users.nama_instansi" class="mb-1" required>
         </b-form-input>
         Bidang Usaha
         <b-form-input v-model="users.bidang_usaha" class="mb-1" required>
         </b-form-input>
         Group Customer
         <b-form-select v-model="users.id_group_customer" class="mb-1" required>
           <option v-for="group in group_customer" :value="group.id_group_customer">
             {{ group.nama_group_customer }}
           </option>
         </b-form-select>
         Jumlah Hari Jatuh Tempo
         <b-form-input v-model="users.jatuh_tempo" type="number" class="mb-1" required>
         </b-form-input>

         <b-button type="submit" variant="success" size="md" class="pull-right">
            <span class="fa fa-check"></span> Simpan
          </b-button>
       </form>
     </b-container>
 </b-modal>
  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data: function(){
      return {
        fields: ["nama","alamat","contact","option"],
        action: "",
        users: {
          id_users: "",
          nama: "",
          alamat: "",
          contact: "",
          email: "",
          username: "",
          password: "",
          image: "",
          status: "",
          id_level: "5",
          require: true,
          margin: 0,
          nama_instansi: "",
          bidang_usaha: "",
          id_group_customer: "",
          jatuh_tempo: 0,
        },
        search : "",
        message: "",
        user: [],
        group_customer: [],
        totalRow: 0,
        perPage: 10,
        currentPage: 1,
        key: "",
        selectedLockPackBarang: [],
        barang: [],
      }
    },

    methods: {
      get_group_customer : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/group_customer", conf)
        .then(response => {
          this.group_customer = response.data.group_customer;
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_barang : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/barang", conf)
        .then(response => {
          this.barang = response.data.barang;
        })
        .catch(error => {
          console.log(error);
        });
      },

      Lock : function(items){
        this.users.id_users = items.id_users;
        this.users.nama = items.nama;
        this.selectedLockPackBarang = [];
        items.lock_pack_barang.forEach((item, i) => {
          let selectedBarang = this.barang.find(it => it.id_barang === item.id_barang);
          this.selectedLockPackBarang.push({
            pack: selectedBarang.pack,
            id_barang: item.id_barang,
            id_pack: item.id_pack,
            harga: item.harga
          })
        });
      },

      bindBarang : function(index){
        let id = this.selectedLockPackBarang[index].id_barang;
        let selectedBarang = this.barang.find(it => it.id_barang === id);
        this.selectedLockPackBarang[index].pack = selectedBarang.pack;
      },

      addLock : function(){
        this.selectedLockPackBarang.push({
          pack: [],
          id_barang: "",
          id_pack: "",
          harga: 0
        })
      },

      dropLock : function(index){
        this.selectedLockPackBarang.splice(index,1);
      },

      SaveLock : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvModal.hide("modal_lock");
        this.$bvToast.show("loading");
        let form = new FormData();
        form.append("id_users", this.users.id_users);
        form.append("lock_pack_barang", JSON.stringify(this.selectedLockPackBarang));
        axios.post(base_url+"/lock-pack-barang", form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.message = response.data.message;
          this.$bvToast.show("message");
          this.find();
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_users : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        axios.get(base_url+"/customer/"+this.perPage+"/"+offset, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.user = response.data.customer;
          this.totalRow = response.data.count;
          this.get_barang();
        })
        .catch(error => {
          console.log(error);
        });
      },

      find : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        let form = new FormData();
        form.append("find", this.search);
        axios.post(base_url+"/customer/"+this.perPage+"/"+offset, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.user = response.data.customer;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      Add : function(){
        this.action = "insert";
        this.users.id_users = "";
        this.users.nama = "";
        this.users.alamat = "";
        this.users.contact = "";
        this.users.email = "";
        this.users.username = "";
        this.users.status = "0";
        this.users.password = "";
        this.users.image = null;
        this.users.require = true;
        this.users.nama_instansi = "";
        this.users.margin = 0;
        this.users.bidang_usaha = "";
        this.users.jatuh_tempo = 0;
        this.users.id_group_customer =
        (this.group_customer.length) ? this.group_customer[0].id_group_customer : "";
      },

      Edit : function(item){
        this.action = "update";
        this.users.id_users = item.id_users;
        this.users.nama = item.nama;
        this.users.alamat = item.alamat;
        this.users.contact = item.contact;
        this.users.email = item.email;
        this.users.username = item.username;
        this.users.status = item.status;
        this.users.password = item.password;
        this.users.image = null;
        this.users.require = false;
        this.users.nama_instansi = item.nama_instansi;
        this.users.margin = item.margin;
        this.users.bidang_usaha = item.bidang_usaha;
        this.users.id_group_customer = item.id_group_customer;
        this.users.jatuh_tempo = item.jatuh_tempo;
      },

      Save : function(){
        this.$bvModal.hide("modal_user");
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("id_users", this.users.id_users);
        form.append("nama", this.users.nama);
        form.append("alamat", this.users.alamat);
        form.append("contact", this.users.contact);
        form.append("email", this.users.email);
        form.append("username", this.users.username);
        form.append("password", this.users.password);
        form.append("id_level", this.users.id_level);
        form.append("status", this.users.status);
        form.append("image", document.getElementById("image").files[0]);
        form.append("margin", this.users.margin);
        form.append("nama_instansi", this.users.nama_instansi);
        form.append("bidang_usaha", this.users.bidang_usaha);
        form.append("jatuh_tempo", this.users.jatuh_tempo);
        form.append("id_group_customer", this.users.id_group_customer);

        let useUrl = "";
        if (this.action === "insert") {
          useUrl = base_url + "/customer/save";
        }else{
          useUrl = base_url + "/customer/update";
        }

        axios.post(useUrl, form, conf)
        .then(response => {
          this.message = response.data.message;
          this.find();
          this.$bvToast.hide("loading");
          this.$bvToast.show("message");
        })
        .catch(error => {
          alert(error);
        })
      },

      Drop : function(id){
        let conf = { headers: { "Api-Token" : this.key} };
        if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {

          this.$bvToast.show("loading");
          axios.delete(base_url + "/customer/drop/"+id, conf)
          .then(response => {
            this.message = response.data.message;
            this.find();
            this.$bvToast.hide("loading");
            this.$bvToast.show("message");
          })
          .catch(error => {
            alert(error);
          })
        }
      },
    },

    mounted(){
      this.get_users();
      this.get_group_customer();
    }
  }
</script>
