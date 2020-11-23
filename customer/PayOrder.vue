<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Pembayaran Tagihan
      </h3>
    </div>
    <div class="card-body">

      <b-alert class="mt-2" variant="warning" :show="bill.length === 0">
        Tidak Ada Tagihan
      </b-alert>

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <form v-on:submit.prevent="pay">
        <div class="mt-2" v-if="bill.length > 0">
          <h4>Verifikasi Setoran Uang: {{ "Rp " + formatNumber(getTotal(bill)) }}</h4>
          <h5 class="mb-1">Upload Bukti Pembayaran</h5>
          <b-form-file v-model="bukti" choose-label="Attach Bukti Pembayaran" required>
          </b-form-file>
          <ul class="mb-3 list-group">
            <li class="list-group-item" v-for="item in bill">
              <b-row>
                <b-col cols="4">
                  <b-form-checkbox v-model="item.status" value="1" unchecked-value="0">
                    <i>{{ "ID Order: " + item.id_orders }}</i>
                 </b-form-checkbox>
                </b-col>
                <b-col cols="4">{{ "Rp " + formatNumber(item.nominal) }}</b-col>
                <b-col cols="4">
                  <b-button class="btn btn-sm btn-success" v-b-modal.modal_detail
                  @click="Detail(item)">
                    <span class="fa fa-eye"></span> Detail Order
                  </b-button>
                </b-col>
              </b-row>
            </li>
          </ul>
        </div>

        <b-button type="submit" class="mt-2" block variant="success"
        v-if="bill.length > 0">
          Verifikasi Setoran Uang
        </b-button>
      </form>
  </div>

  <b-modal id="modal_detail" title="Detail Order"
    header-bg-variant="info" size="lg"
    border-variant="info" hide-footer>

    <b-container fluid>
      <div class="row mb-3">
        <div class="col-6">
          <h4>ID Order: {{ order.id_orders }}</h4>
          <h4>PO: {{ order.po }}</h4>
          <h4>Invoice: {{ order.invoice }}</h4>
          <h4>Customer: {{ order.nama_pembeli }}</h4>
        </div>
        <div class="col-6">
          <h4>Tgl. Order: {{ formatDateTime(order.waktu_order) }}</h4>
          <h4>Tgl. Pengiriman: {{ formatDate(order.waktu_pengiriman) }}</h4>
          <h4>Tgl. Jatuh Tempo: {{ formatDate(order.tgl_jatuh_tempo) }}</h4>
        </div>
      </div>
      <div class="row mb-3">
        <ul class="list-group col-12">
          <li class="list-group-item mb-2" v-for="d in detail_orders">
            <div class="row mb-2">
              <div class="col-3">
                {{ d.barang.nama_barang }}
              </div>
              <div class="col-2">
                {{ d.jumlah_barang + " Kg" }}
              </div>
              <div class="col-3">
                {{ "@ Rp " + formatNumber(d.harga_beli) }}
              </div>
              <div class="col-4">
                {{ "Rp " + formatNumber(d.harga_beli * d.jumlah_barang) }}
              </div>
            </div>
            Pack
            <div class="row mb-2">
              <div class="col-3">
                {{ d.pack.nama_pack }}
              </div>
              <div class="col-2">
                {{ d.jumlah_pack + " item" }}
              </div>
              <div class="col-3" v-if="(d.harga_pack > 0)">
                {{ "@ Rp " + formatNumber(d.harga_pack) }}
              </div>
              <div class="col-4" v-if="(d.harga_pack > 0)">
                {{ "Rp " + formatNumber(d.harga_pack * d.jumlah_pack) }}
              </div>
            </div>
          </li>
          <li class="list-group-item mb-2">
            <div class="row">
              <div class="col-8">

              </div>
              <div class="col-4">
                {{ "Rp " + formatNumber(order.total_bayar) }}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </b-container>
  </b-modal>
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
      }
    },

    methods: {
      Detail : function(item){
        this.order.id_orders = item.id_orders;
        this.order.po = item.po;
        this.order.invoice = item.invoice;
        this.order.nama_pembeli = item.pembeli.nama;
        this.order.waktu_order = item.waktu_order;
        this.order.waktu_pengiriman = item.waktu_pengiriman;
        this.order.tgl_jatuh_tempo = item.tgl_jatuh_tempo;
        this.order.total_bayar = item.total_bayar;
        this.detail_orders = item.detail_orders;
      },

      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          if(item.status === "1")
          total += Number(item.nominal)
        });
        return total;
      },

      get_bill : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/customer-bill/" + this.users.id_users, conf)
        .then(response => {
          response.data.forEach((item, i) => {
            item.status = "0";
          });
          this.bill = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/customer/check", form)
        .then(response => {
          if (response.data.auth == false) {
            window.location = "login.html";
          } else{
            this.users = response.data.customer;
            this.get_bill();
          }
        })
        .catch(error => {
          console.log(error);
        });
      },

      pay : function(){
        if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
          this.bill.forEach((item, i) => {
            if (item.status === "1"){
              this.selectedBill.push({
                id_orders: item.id_orders,
                nominal: item.nominal
              })
            }
          });
          if (this.selectedBill.length == 0) {
            this.$bvToast.show("loading");
            let conf = { headers: { "Api-Token" : this.key} };
            let form = new FormData();
            form.append("pembayaran_orders", JSON.stringify(this.selectedBill));
            form.append("bukti", this.bukti);
            form.append("id_users", this.users.id_users);
            axios.post(base_url + "/pay-orders", form, conf)
            .then(response => {
              this.$bvToast.hide("loading");
              this.message = response.data.message;
              this.$bvToast.show("message");
              this.get_bill();
            })
            .catch(error => {
              console.log(error);
            })
          }else{
            alert("Anda belum memilih order yang dibayarkan!")
          }

        }
      },
    },

    mounted(){
      this.initUser();
    }
  }
</script>
