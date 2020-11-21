<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-truck-pickup"></span>
        Data Sopir
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
 <b-modal
     id="modal_user"
     title="Form Sopir"
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
          id_level: "4",
          require: true
        },
        search : "",
        message: "",
        user: [],
        totalRow: 0,
        perPage: 10,
        currentPage: 1,
        key: ""
      }
    },

    methods: {
      get_users : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        axios.get(base_url+"/driver/"+this.perPage+"/"+offset, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.user = response.data.driver;
          this.totalRow = response.data.count;
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
        axios.post(base_url+"/driver/"+this.perPage+"/"+offset, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.user = response.data.driver;
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

        let useUrl = "";
        if (this.action === "insert") {
          useUrl = base_url + "/driver/save";
        }else{
          useUrl = base_url + "/driver/update";
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

          axios.delete(base_url + "/driver/drop/"+id, conf)
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
    }
  }
</script>
