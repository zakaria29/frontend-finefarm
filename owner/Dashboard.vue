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
                    Update Harga <strong class="text-info">{{ currentUpdate }}</strong> 
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

    <div class="card-footer">
      <h2>Grafik Omset</h2>
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
        hargaBarang: [],
        stokBarang: [],
        pengguna: [],
        currentUpdate: "",
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
        axios.post(base_url+"/owner/dashboard", form)
        .then(response => {
          this.users = response.data.users;
          this.total_orders = response.data.total_orders;
          this.hargaBarang = response.data.harga_barang;
          this.stokBarang = response.data.stok_barang;
          this.pengguna = response.data.pengguna;
          this.currentUpdate = formatDate(response.data.currentUpdate);
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
