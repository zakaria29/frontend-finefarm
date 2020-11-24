<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Verifikasi Pembayaran
      </h3>
    </div>
    <div class="card-body">

      <b-alert class="mt-2" variant="warning" :show="pay.length === 0">
        Tidak Ada Pembayaran yang perlu diverifikasi
      </b-alert>

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <div class="mt-2" v-if="pay.length > 0">
        <ul class="mb-3 list-group">
          <li class="list-group-item" v-for="item in pay">
            <b-row>
              <b-col cols="4">
                <h5>{{ "ID Orders: " + item.id_orders }}</h5>
                <i>{{ "By: " + item.users.nama }}</i>
              </b-col>
              <b-col cols="4">
                {{ "Rp " + formatNumber(item.nominal) }} <br />
                <small> <i>Waktu: {{ formatDateTime(item.waktu) }}</i> </small>
              </b-col>
              <b-col cols="4">
                <b-row class="mb-1">
                  <b-col>
                    <b-button size="sm" variant="success" block @click="verifyPay(item,'2')">
                      <span class="fa fa-check"></span> Valid
                    </b-button>
                  </b-col>
                  <b-col>
                    <b-button size="sm" variant="danger" block @click="verifyPay(item,'1')">
                      <span class="fa fa-window-close"></span> Invalid
                    </b-button>
                  </b-col>
                </b-row>
                <b-row class="mb-1">
                  <b-col>
                    <b-button block class="btn btn-sm btn-warning" v-b-modal.modal_detail
                    @click="Detail(item.orders)">
                      <span class="fa fa-eye"></span> Detail Order
                    </b-button>
                  </b-col>
                  <b-col>
                    <b-button block class="btn btn-sm btn-info" v-b-modal.modal_preview
                    @click="previewFile(item)">
                      <span class="fa fa-eye"></span> Lihat Bukti
                    </b-button>
                  </b-col>
                </b-row>
              </b-col>
            </b-row>
          </li>
        </ul>
      </div>
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
  <b-modal id="modal_preview"
    size="xl"
    :title="file_title"
    header-bg-variant="info"
    header-text-variant="white"
    border-variant="info" hide-footer>

    <iframe :src="source_file" width="100%" height="450"></iframe>
  </b-modal>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        pay: [],
        message : "",
        source_file: null,
        file_title: null,
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

      previewFile: function(item){
        this.file_title = item.id_pay;
        let url = new URL(item.bukti);
        let id = url.searchParams.get("id");
        this.source_file = "https://drive.google.com/file/d/"+id+"/preview";
      },

      get_pay : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/verify-pembayaran", conf)
        .then(response => {
          this.pay = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      verifyPay : function(item, ket){
        if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("id_pay", item.id_pay);
          form.append("keterangan", ket);
          axios.post(base_url + "/verify-pembayaran", form, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.get_pay();
          })
          .catch(error => {
            console.log(error);
          })
        }
      },
    },

    mounted(){
      this.get_pay();
    }
  }
</script>
