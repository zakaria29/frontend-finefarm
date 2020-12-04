<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-home"></span> Home
      </h3>
    </div>
    <div class="card-body" v-if="users !== null">
      <div class="row">
        <div class="col-xl-4 col-md-6">
          <div class="card">
            <div class="card-header">
              <div class="row align-items-center">
                <div class="col">
                  <h3 class="mb-0">
                    <a class="text-primary" v-b-modal.modal_edit @click="EditProfil">
                      <span class="fa fa-edit"></span>
                    </a>
                    Profil Customer
                  </h3>
                </div>
              </div>
            </div>
            <div class="card-body">
              <b-row class="my-2 justify-content-center">
                <img :src="storage_path + users.image"
                width="100" height="100"
                class="rounded-circle">
              </b-row>
              <b-row>
                <b-col cols="3">Nama</b-col>
                <b-col>: {{ users.nama }}</b-col>
              </b-row>
              <b-row>
                <b-col cols="3">Alamat</b-col>
                <b-col>: {{ users.alamat }}</b-col>
              </b-row>
              <b-row>
                <b-col cols="3">Kontak</b-col>
                <b-col>: {{ users.contact }}</b-col>
              </b-row>
              <b-row>
                <b-col cols="3">Email</b-col>
                <b-col>: {{ users.email }}</b-col>
              </b-row>
            </div>
          </div>
        </div>

        <div class="col-xl-4 col-md-6">
          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">Total Order</h5>
                  <span class="h2 font-weight-bold mb-0">{{ users.orders.length }}</span>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-success text-white rounded-circle shadow">
                    <i class="ni ni-shop"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Total Nominal Order
                  </h5>
                  <span class="h2 font-weight-bold mb-0">
                    {{ "Rp " + formatNumber(getTotal(users.orders)) }}
                  </span>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-info text-white rounded-circle shadow">
                    <i class="fa fa-money-bill-alt"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Tanggungan Pembayaran
                  </h5>
                  <span class="h2 font-weight-bold mb-0">
                    {{ "Rp " + formatNumber(users.tanggungan_pembayaran.nominal) }}
                  </span>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-danger text-white rounded-circle shadow">
                    <i class="ni ni-money-coins"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </div>

        <div class="col-xl-4 col-md-6">
          <!-- tanggungan pack -->
          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Tanggungan Pack
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <b-row v-for="p in users.tanggungan_pack">
                      <b-col cols="7">{{ p.pack.nama_pack }}</b-col>
                      <b-col>: {{ p.jumlah }} <small>item</small> </b-col>
                    </b-row>
                  </h4>

                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-warning text-white rounded-circle shadow">
                    <i class="ni ni-basket"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <b-toast id="message" title="Message" solid>
          <strong class="text-success">{{ message }}</strong>
        </b-toast>

        <b-toast id="loading" title="Processing Data" solid no-auto-hide>
          <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
          <span class="fa fa-spinner fa-spin"></span>
          <strong class="text-success">Loading...</strong>
        </b-toast>

        <b-modal id="modal_edit" title="Detail Order"
          header-bg-variant="info" size="sm"
          border-variant="info" hide-footer>
          <form v-on:submit.prevent="saveProfil">
            <b-container fluid>
              Nama
              <b-form-input v-model="profile.nama" required class="mb-1">
              </b-form-input>
              Alamat
              <b-form-input v-model="profile.alamat" required class="mb-1">
              </b-form-input>
              Email
              <b-form-input v-model="profile.email" type="email" required class="mb-1">
              </b-form-input>
              Kontak
              <b-form-input v-model="profile.contact" required class="mb-1">
              </b-form-input>
              Username
              <b-form-input v-model="profile.username" required class="mb-1">
              </b-form-input>
              <b-form-checkbox v-model="profile.changePassword">
                Ubah Password
              </b-form-checkbox>
              <div v-if="profile.changePassword">
                Password
                <b-form-input v-model="profile.password" type="password" required class="mb-1">
                </b-form-input>
              </div>

              <b-button type="submit" block variant="primary">
                Simpan
              </b-button>
            </b-container>
          </form>
        </b-modal>

      </div>
    </div>
  </div>

</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        bill: [],
        bukti: null,
        selectedBill: [],
        message : "",
        key: "",
        order : {
          id_orders : "",
          nama_pembeli: "",
          invoice: "",
          po: "",
          waktu_order: "",
          waktu_pengiriman: "",
          tgl_jatuh_tempo: "",
          total_bayar: 0,
        },
        detail_orders: [],
        profile: {
          nama: "",
          alamat: "",
          email: "",
          contact: "",
          username: "",
          password: "",
          changePassword: false
        }
      }
    },

    methods: {
      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          if (item.id_status_orders >= 3) {
            total += Number(item.total_bayar)
          }

        });
        return total;
      },

      EditProfil : function(){
        this.profile.nama = this.users.nama;
        this.profile.alamat = this.users.alamat;
        this.profile.contact = this.users.contact;
        this.profile.email = this.users.email;
        this.profile.username = this.users.username;
        this.profile.password = "";
        this.profile.changePassword = false;
      },

      saveProfil : function(){
        this.$bvToast.show("loading");
        this.$bvModal.hide("modal_edit");
        let form = new FormData();
        form.append("id_users", this.users.id_users);
        form.append("nama", this.profile.nama);
        form.append("alamat", this.profile.alamat);
        form.append("email", this.profile.email);
        form.append("contact", this.profile.contact);
        form.append("username", this.profile.username);
        if (this.profile.changePassword) {
          form.append("password", this.profile.password);
        }

        axios.post(base_url + "/cashier/profil", form)
        .then(response => {
          this.$bvToast.hide("loading");
          this.message = response.data.message;
          this.$bvToast.show("message");
          this.initUser();
        })
        .catch(error => console.log(error))
      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/customer/dashboard", form)
        .then(response => {
          this.users = response.data;
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
