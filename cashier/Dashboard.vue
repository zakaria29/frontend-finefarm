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
                  <h3 class="mb-0">Profil Cashier</h3>
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
              <router-link to="/list_order">
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
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/accept">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Order yang perlu diverifikasi
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ total_verify }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-info text-white rounded-circle shadow">
                      <i class="ni ni-check-bold"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/send">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Order yang perlu dikirim
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ total_send }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-danger text-white rounded-circle shadow">
                      <i class="ni ni-delivery-fast"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

        </div>

        <div class="col-xl-4 col-md-6">
          <!-- tanggungan pack -->
          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/setoran-pack">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Verifikasi Setoran Pack
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
                    <div class="icon icon-shape bg-warning text-white rounded-circle shadow">
                      <i class="ni ni-basket"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/verify-pembayaran">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Pembayaran yang perlu diverifikasi
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ "Rp " + formatNumber(pay_verify) }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-cyan text-white rounded-circle shadow">
                      <i class="ni ni-money-coins"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/setoran-uang">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Setoran uang yang perlu diverifikasi
                    </h5>
                    <span class="h2 font-weight-bold mb-0">
                      {{ "Rp " + formatNumber(setor_uang) }}
                    </span>
                  </div>
                  <div class="col-auto">
                    <div class="icon icon-shape bg-primary text-white rounded-circle shadow">
                      <i class="fa fa-money-bill-alt"></i>
                    </div>
                  </div>
                </div>
              </router-link>
            </div>
          </div>

          <div class="card card-stats">
            <div class="card-body">
              <router-link to="/verify-kembali-orders">
                <div class="row">
                  <div class="col">
                    <h5 class="card-title text-uppercase text-muted mb-0">
                      Verifikasi Pengurangan Order
                    </h5>
                    <h4 class="font-weight-bold mb-0">
                      <div v-for="b in kembali_orders">
                        <b-row v-for="ko in b.detail_kembali_orders">
                          <b-col cols="9">
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
              </router-link>
            </div>
          </div>

        </div>

      </div>

    </div>

    <div class="card-footer">
      <h2>Grafik Order</h2>
      <form v-on:submit.prevent="generateChart">
        <b-row class="mb-2">
          <b-col cols="5">
            Start Date
             <b-form-input v-model="from" type="date" required></b-form-input>
          </b-col>
          <b-col cols="5">
            End Date
            <b-form-input v-model="to" type="date" required></b-form-input>
          </b-col>
          <b-col cols="2">
            <br />
            <b-button variant="primary" block type="submit">
              Show
            </b-button>
          </b-col>
        </b-row>
      </form>

      <div class="row">
        <b-col>
          <canvas ref="chLine" height="100"></canvas>
        </b-col>
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
        from: "",
        to: "",
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

      generateChart : function(){
        let colors = ['#007bff','#28a745','#333333','#c3e6cb','#dc3545','#6c757d'];
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("from", this.from);
        form.append("to", this.to);
        axios.post(base_url + "/grafik", form)
        .then(response => {
          let lbl = [];
          let ds = [];
          response.data.forEach((item, i) => {
            lbl.push(item.waktu);
            ds.push(item.total);
          });

          let chartData = {
            labels: lbl,
            datasets: [{
              data: ds,
              backgroundColor: 'transparent',
              borderColor: colors[0],
              borderWidth: 4,
              pointBackgroundColor: colors[0]
            }]
          };

          let chLine = this.$refs.chLine;

          if (chLine) {
            new Chart(chLine.getContext('2d'), {
            type: 'line',
            data: chartData,
            options: {
              scales: {
                yAxes: [{
                  ticks: {
                    callback : function(label, index, labels){
                      return formatNumber(label);
                    }
                  }
                }],
                xAxes: [{
                  ticks: {
                    callback : function(label, index, labels){
                      return dateFormat(label);
                    }
                  }
                }]
              },
              legend: {
                display: false
              },
              tooltips: {
                callbacks: {
                  label: function(tooltipItem, data){
                    return tooltipItem.yLabel.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
                  }
                }
              },
            }
            });
          }
        })
        .catch(error => console.log(error))

      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/cashier/dashboard", form)
        .then(response => {
          this.users = response.data.users;
          this.total_orders = response.data.total_orders;
          this.pay_verify = response.data.pay_verify;
          this.total_send = response.data.total_send;
          this.total_verify = response.data.total_verify;
          this.kembali_pack = response.data.kembali_pack;
          this.setor_uang = response.data.setor_uang;
          this.kembali_orders = response.data.kembali_orders;

          this.generateChart();
        })
        .catch(error => {
          console.log(error);
        });
      },


    },

    mounted(){
      let date = new Date();
      let currentDate = date.toISOString().slice(0,10);
      this.to = currentDate;
      let lastMonth = new Date(date.getFullYear(), date.getMonth()-1, date.getDate());
      this.from = lastMonth.toISOString().slice(0,10);
      this.initUser();
    }
  }
</script>
