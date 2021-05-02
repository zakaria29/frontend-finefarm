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
                    Profil Driver
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
            <div :class="kirim_order > 0 ? 'card-body bg-lighter' : 'card-body'">
              <router-link to="/deliver">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Order yang akan dikirim
                    </h5>
                    <span class="h2 font-weight-bold mb-0">{{ kirim_order }}</span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-warning text-white rounded-circle shadow">
                      <i class="ni ni-delivery-fast"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div :class="delivered_order > 0 ? 'card-body bg-lighter' : 'card-body'">
              <router-link to="/delivered">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Penerimaan Order
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ delivered_order }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-success text-white rounded-circle shadow">
                      <i class="fa fa-people-carry"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div :class="prepare_barang.filter(it => it.log_get_supplier.length > 0) > 0 ?
            'card-body bg-lighter' : 'card-body'">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Barang yang harus disiapkan
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in prepare_barang">
                      <b-row v-for="lg in b.log_get_supplier">
                        <b-col cols="8">
                          {{ b.nama_barang }}
                          <small class="text-warning">
                            ({{ lg.supplier.nama_supplier }})
                          </small>
                        </b-col>
                        <b-col>: {{ lg.jumlah }}
                          <small>{{ (b.satuan === "1") ? " Kg" : " Butir" }}</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>

                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-blue text-white rounded-circle shadow">
                    <i class="ni ni-box-2"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="card card-stats">
            <div :class="prepare_pack.filter(it => it.detail_orders.length > 0) > 0 ?
            'card-body bg-lighter' : 'card-body'">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Pack yang harus disiapkan
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="p in prepare_pack">
                      <b-row v-if="p.detail_orders.length > 0">
                        <b-col cols="8">{{ p.nama_pack }}</b-col>
                        <b-col>: {{ p.detail_orders[0].jumlah }} <small>item</small> </b-col>
                      </b-row>
                    </div>
                  </h4>

                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-info text-white rounded-circle shadow">
                    <i class="ni ni-basket"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="card card-stats">
            <div :class="retur_order.barang.filter(it => it.detail_retur_order.length > 0) > 0 ?
            'card-body bg-lighter' : 'card-body'">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Barang yang di retur
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in retur_order.barang">
                      <b-row v-for="ko in b.detail_retur_order">
                        <b-col cols="8">
                          {{ b.nama_barang }}
                        </b-col>
                        <b-col>: {{ ko.jumlah }}
                          <small>{{ (b.satuan === "1") ? " Kg" : " Butir" }}</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>
                  <hr />
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Pack yang di retur
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="k in retur_order.pack">
                      <b-row v-for="ko in k.detail_retur_order">
                        <b-col cols="8">
                          {{ k.nama_pack }}
                        </b-col>
                        <b-col>: {{ ko.jumlah }}
                          <small> item</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-dark text-white rounded-circle shadow">
                    <i class="ni ni-box-2"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </div>

        <div class="col-xl-4 col-md-6">

          <div class="card card-stats">
            <div :class="setor_uang > 0 ? 'card-body bg-lighter' : 'card-body'">
              <router-link to="/setoran-uang">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Uang yang harus disetorkan
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ "Rp " + formatNumber(setor_uang) }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-danger text-white rounded-circle shadow">
                      <i class="fa fa-money-bill-alt"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <!-- tanggungan pack -->
          <div class="card card-stats">
            <div :class="kembali_pack.filter(it => it.kembali_pack.length > 0) > 0 ?
            'card-body bg-lighter' : 'card-body'">
              <router-link to="/setoran-pack">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Pack yang harus disetorkan
                    </h5>
                    <h4 class="font-weight-bold mb-0">
                      <div v-for="p in kembali_pack">
                        <b-row v-if="p.kembali_pack.length > 0">
                          <b-col cols="8">{{ p.nama_pack }}</b-col>
                          <b-col>: {{ p.kembali_pack[0].jumlah }} <small>item</small> </b-col>
                        </b-row>
                      </div>
                    </h4>

                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-danger text-white rounded-circle shadow">
                      <i class="ni ni-basket"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div :class="kembali_orders.barang.filter(it => it.detail_kembali_orders.length > 0) > 0 ?
            'card-body bg-lighter' : 'card-body'">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Barang yang harus dikembalikan
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in kembali_orders.barang">
                      <b-row v-for="ko in b.detail_kembali_orders">
                        <b-col cols="8">
                          {{ b.nama_barang }}
                        </b-col>
                        <b-col>: {{ ko.jumlah }}
                          <small>{{ (b.satuan === "1") ? " Kg" : " Butir" }}</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>
                  <hr />
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Pack yang harus dikembalikan
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="k in kembali_orders.pack">
                      <b-row v-for="ko in k.detail_kembali_orders">
                        <b-col cols="8">
                          {{ k.nama_pack }}
                        </b-col>
                        <b-col>: {{ ko.jumlah }}
                          <small> item</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-dark text-white rounded-circle shadow">
                    <i class="ni ni-box-2"></i>
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
        kembali_pack: [],
        setor_uang: 0,
        kirim_order: 0,
        delivered_order: 0,
        prepare_barang: [],
        prepare_pack: [],
        kembali_orders: [],
        retur_order: [],
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
        axios.post(base_url+"/driver/dashboard", form)
        .then(response => {
          this.users = response.data.users;
          this.kembali_pack = response.data.kembali_pack;
          this.setor_uang = response.data.setor_uang;
          this.kirim_order = response.data.kirim_order;
          this.delivered_order = response.data.delivered_order;
          this.prepare_pack = response.data.prepare_pack;
          this.prepare_barang = response.data.prepare_barang;
          this.kembali_orders = response.data.kembali_orders;
          this.retur_order = response.data.retur_order;
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
