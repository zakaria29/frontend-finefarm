<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Tanggungan Pembayaran
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

      <b-row class="mb-1">
        <b-col>
          <form v-on:submit.prevent="getTanggungan">
             <b-form-input v-model="search" class="mb-1" placeholder="Pencarian...">
             </b-form-input>
          </form>
        </b-col>
      </b-row>
      <b-alert class="mt-2" variant="warning" :show="tanggungan.length === 0">
        Tidak Ada Tanggungan Pembayaran
      </b-alert>

      <h2> <strong>Total Piutang: Rp {{ formatNumber(total) }} </strong> </h2>

      <div class="mt-2" v-if="tanggungan.length > 0">
        <ul class="list-group">
          <li class="list-group-item" v-for="item in tanggungan" v-if="item.bill.length > 0">
            <h3>{{ item.nama }}</h3>
            <h4 class="text-primary">
              Tanggungan Pembayaran: {{ "Rp "+ formatNumber(getTotal(item.bill)) }}
            </h4>
            <h5 class="text-success">Rincian</h5>
            <b-row>
              <b-col cols="6" v-for="it in item.bill">
                <strong class="text-info">{{ it.id_orders }}</strong>
                <strong>: {{ "Rp " + formatNumber(it.nominal) }} </strong><br />
                <b-badge pill variant="info" v-if="it.status === '0'">
                  Menunggu Verifikasi
                </b-badge>
              </b-col>
            </b-row>
          </li>
        </ul>
      </div>
  </div>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        pack: [],
        tanggungan: [],
        message : "",
        search: "",
        key: "",
        selectedUser: null,
        tanggungan_pack: [],
        total: 0,
      }
    },

    methods: {
      getTotal : function(item){
        let total = 0;
        item.forEach((it, i) => {
          total += Number(it.nominal)
        });
        return total;
      },
      getTanggungan : function(){
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("find", this.search);

        axios.post(base_url + "/customer-tagihan", form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.tanggungan = response.data;
          response.data.forEach((item, i) => {
            item.bill.forEach((it, j) => {
              this.total = Number(this.total) + Number(it.nominal)
            });
          });

        })
        .catch(error => {
          console.log(error);
        })
      },

      get_pack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/pack", conf)
        .then(response => {
          this.pack = response.data.pack;
          this.getTanggungan();
        })
        .catch(error => {
          console.log(error);
        })
      },

      init : async function(){
        await this.getTanggungan();
      },
    },

    mounted(){
      this.init();
    }
  }
</script>
