<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Order
      </h3>
    </div>
    <div class="card-body">
      <b-form-input v-model="search" @keyup.enter="find(users.id_users)"
      placeholder="Pencarian...">
      </b-form-input>
      <ul class="list-group mt-2">
        <li class="list-group-item mb-2" v-for="item in orders">
          <b-row>
            <b-col cols="5">
              <h4>ID Order: {{ item.id_orders }}</h4>
              <h4>Waktu Order: {{ formatDateTime(item.waktu_order) }}</h4>
              <h4>Waktu Kirim: {{ formatDate(item.waktu_pengiriman) }}</h4>
              <b-row>
                <b-col>
                  <h3 class="text-info">
                    Total: Rp {{ formatNumber(item.total_bayar) }}

                    <b-badge pill variant="success"
                    v-if="item.tagihan !== null && item.tagihan.status === '2'">
                      <strong>Lunas</strong>
                    </b-badge>
                    <b-badge pill variant="warning"
                    v-if="item.tagihan !== null && item.tagihan.status === '0'">
                      <strong>Menunggu Verifikasi Pembayaran</strong>
                    </b-badge>
                    <b-badge pill variant="danger"
                    v-if="item.tagihan !== null && item.tagihan.status === '1'">
                      <strong>Belum Lunas</strong>
                    </b-badge>
                  </h3>
                </b-col>
              </b-row>
            </b-col>
            <b-col cols="5">
              <h4>Customer: {{ item.pembeli.nama }}</h4>
              <h4>Kasir: {{ item.users === null ? "-" : item.users.nama }}</h4>
              <h4>Sopir: {{ item.sopir === null ? "-" : item.sopir.nama }}</h4>
            </b-col>
            <b-col>
              <b-button size="sm" class="btn btn-block btn-sm btn-info" v-if="(item.id_status_orders < 2)"
              @click="Edit(item.id_orders)">
                <span class="fa fa-edit"></span> Edit Order
              </b-button>

              <b-button class="btn btn-block btn-sm btn-success" v-b-modal.modal_detail
              @click="Detail(item)">
                <span class="fa fa-eye"></span> Detail Order
              </b-button>

              <b-button class="btn btn-block btn-sm btn-warning" v-b-modal.modal_log
              @click="LogOrder(item)">
                <span class="fa fa-history"></span> Log Order
              </b-button>
            </b-col>
          </b-row>
          <b-row class="my-1" v-if="item.catatan !== ''">
            <b-col>
              Catatan: {{ item.catatan }}
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              Status Order:
              <b-badge pill class="text-dark" :variant="type[item.id_status_orders]">
                {{ item.status_orders.nama_status_order }}
              </b-badge>
            </b-col>
          </b-row>
          <b-row class="my-1" v-if="item.kendala !== ''">
            <b-col>
              Kendala Pengiriman:
              <b-badge class="bg-orange text-white">
                {{ item.kendala }}
              </b-badge>
            </b-col>
          </b-row>
        </li>
      </ul>
      <b-pagination
      size="md"
      class="mt-3"
      :total-rows="totalRow"
      v-model="currentPage"
      :per-page="perPage"
      align="center"
      v-on:input="find(users.id_users)">
      </b-pagination>
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

    <b-modal id="modal_log" title="Log Order"
      header-bg-variant="success" size="lg"
      border-variant="info" hide-footer>

      <ul class="timeline">
        <li v-for="l in log_orders">
					<span class="text-default">{{ l.users.nama }}</span>
					<span class="text-warning float-right">
            <small>{{ formatDateTime(l.waktu) }}</small>
          </span>
					<p>
            {{ l.status_orders.nama_status_order }}
          </p>
				</li>
      </ul>
    </b-modal>
  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        message : "",
        key: "",
        orders: [],
        detail_orders: [],
        log_orders: [],
        totalRow: 0,
        perPage: 10,
        currentPage: 1,
        from: "",
        to: "",
        search: "",
        fields: ["waktu_order","nama_pembeli","nominal","option"],
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
      }
    },

    methods: {
      Edit : function(id){
        this.$router.push({ name: "EditOrder", params: { id_orders: id }});

      },
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

      LogOrder : function(item){
        this.log_orders = item.log_orders;
      },

      find : function(id){
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        // form.append("from", this.from);
        // form.append("to", this.to);
        form.append("find", this.search);
        let offset = (this.currentPage-1) * this.perPage;

        axios.post(base_url + "/customer-orders/" + id + "/" +
        this.perPage + "/" + offset, form, conf)
        .then(response => {
          this.orders = response.data.orders;
          this.totalRow = response.data.count;
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
            this.find(response.data.customer.id_users);
          }
        })
        .catch(error => {
          console.log(error);
        });
      },
    },

    mounted(){
      this.initUser();
      // let date = new Date();
      // let currentDate = date.toISOString().slice(0,10);
      // this.to = currentDate;
      // let lastMonth = new Date(date.getFullYear(), date.getMonth()-1, date.getDate());
      // this.from = lastMonth.toISOString().slice(0,10);
    }
  }
</script>
