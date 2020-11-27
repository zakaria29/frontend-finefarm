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
                  <h3 class="mb-0">Profil Customer</h3>
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
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
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
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
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
            <div class="card-body">
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

        </div>

        <div class="col-xl-4 col-md-6">

          <div class="card card-stats">
            <div class="card-body">
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
            </div>
          </div>

          <!-- tanggungan pack -->
          <div class="card card-stats">
            <div class="card-body">
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
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Barang yang harus dikembalikan
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in kembali_orders">
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
