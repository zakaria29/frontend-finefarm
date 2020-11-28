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
                  <h3 class="mb-0">Profil Owner</h3>
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
                  <span class="h2 font-weight-bold mb-0">{{ total_orders }}</span>
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
                    Jumlah Pengguna
                  </h5>
                  <div v-for="u in pengguna">
                    <b-row>
                      <b-col cols="6">{{ u.role }}</b-col>
                      <b-col>: {{ u.count + " orang" }}</b-col>
                    </b-row>
                  </div>
                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-info text-white rounded-circle shadow">
                    <i class="fa fa-users"></i>
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
                    Harga Barang Terbaru
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in hargaBarang">
                      <b-row v-if="b.current_harga !== null">
                        <b-col cols="6">{{ b.nama_barang }}</b-col>
                        <b-col>: {{ "Rp " + formatNumber(b.current_harga.harga) }}</b-col>
                      </b-row>
                    </div>
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

          <div class="card card-stats">
            <div class="card-body">
              <div class="row">
                <div class="col">
                  <h5 class="card-title text-uppercase text-muted mb-0">
                    Stok Barang
                  </h5>
                  <h4 class="font-weight-bold mb-0">
                    <div v-for="b in stokBarang">
                      <b-row v-if="b.stok.length > 0">
                        <b-col cols="6">{{ b.nama_barang }}</b-col>
                        <b-col>
                          : {{ b.stok[0].stok }}
                          <small>{{ b.satuan === "1" ? " Kg" : " Butir" }}</small>
                        </b-col>
                      </b-row>
                    </div>
                  </h4>

                </div>
                <div class="col-auto">
                  <div class="icon icon-shape bg-success text-white rounded-circle shadow">
                    <i class="ni ni-basket"></i>
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
        total_orders: 0,
        total_verify: 0,
        total_send: 0,
        pay_verify: 0,
        kembali_pack: 0,
        setor_uang: 0,
        kembali_orders: [],

        hargaBarang: [],
        stokBarang: [],
        pengguna: [],
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
        axios.post(base_url+"/owner/dashboard", form)
        .then(response => {
          this.users = response.data.users;
          this.total_orders = response.data.total_orders;
          this.hargaBarang = response.data.harga_barang;
          this.stokBarang = response.data.stok_barang;
          this.pengguna = response.data.pengguna;
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
