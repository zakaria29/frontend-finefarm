<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Order Siap Kirim
      </h3>
    </div>
    <div class="card-body">
      <form v-on:submit.prevent="find">
        <div class="row mb-2">
          <div class="col-10">
            Pencarian
             <b-form-input v-model="search" type="text" placeholder="Kata Kunci">
             </b-form-input>
          </div>
          <div class="col-2">
            <br />
            <b-button type="submit" class="btn btn-info">
              <span class="fa fa-search"></span> Cari
            </b-button>
          </div>
        </div>
      </form>

      <ul class="list-group mt-2">
        <li class="list-group-item mb-2" v-for="item in orders">
          <b-row>
            <b-col cols="5">
              <h4>ID Order: {{ item.id_orders }}</h4>
              <h4>Waktu Order: {{ formatDateTime(item.waktu_order) }}</h4>
              <h4>Waktu Kirim: {{ formatDate(item.waktu_pengiriman) }}</h4>
              <h3 class="text-info">Total: Rp {{ formatNumber(item.total_bayar) }}</h3>
            </b-col>
            <b-col cols="5">
              <h4>Customer: {{ item.pembeli.nama }}</h4>
              <h4>Kasir: {{ item.users === null ? "-" : item.users.nama }}</h4>
              <h4>Sopir: {{ item.sopir === null ? "-" : item.sopir.nama }}</h4>
            </b-col>
            <b-col>
              <b-button class="btn btn-block btn-sm btn-info"
              @click="SendOrder(item)">
                <span class="fa fa-truck"></span> Kirim Order
              </b-button>

              <!-- <b-button class="btn btn-block btn-sm btn-success" v-b-modal.modal_detail
              @click="Detail(item)">
                <span class="fa fa-eye"></span> Detail Order
              </b-button> -->

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

          <b-row class="my-2">
            <b-col>
              <ul class="list-group">
                <li class="list-group-item" v-for="it in item.detail_orders">
                  <b-row>
                    <b-col cols="3">
                      <small class="text-info">Nama Barang</small>
                      <h4>{{ it.barang.nama_barang }}</h4>
                    </b-col>
                    <b-col cols="3">
                      <small class="text-info">Qty</small>
                      <h4>
                        {{ it.jumlah_barang }} {{ it.barang.satuan === '1' ? 'Kg' : 'Butir' }} <br />
                        <i class="text-primary">[{{ it.pack.nama_pack }} : {{ it.jumlah_pack }} item]</i>
                      </h4>
                    </b-col>
                    <b-col cols="3">
                      <small class="text-info">Harga</small>
                      <h4>{{ "Rp " + formatNumber(it.harga_beli) }}</h4>
                    </b-col>
                    <b-col cols="3">
                      <small class="text-info">Total</small>
                      <h4>{{ "Rp " + formatNumber(it.harga_beli * it.jumlah_barang) }}</h4>
                    </b-col>

                  </b-row>
                </li>
              </ul>
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
      v-on:input="find">
      </b-pagination>
    </div>

    <b-toast id="message" title="Message" solid>
      <strong class="text-success">{{ message }}</strong>
    </b-toast>

    <b-toast id="loading" title="Processing Data" solid no-auto-hide>
      <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
      <span class="fa fa-spinner fa-spin"></span>
      <strong class="text-success">Loading...</strong>
    </b-toast>

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
					<span class="text-default">{{ l.nama }}</span>
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

      SendOrder: function(item){
        if (confirm("Apakah Anda yakin akan mengirim order ini?")) {
          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let id_orders = item.id_orders;
          let id_users = this.users.id_users;
          axios.get(base_url + "/deliver-orders/" + id_orders + "/" + id_users, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.find();
          })
          .catch(error => {
            alert(error);
          })
        }
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

      find : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("find", this.search);
        let offset = (this.currentPage-1) * this.perPage;
        let id_sopir = this.users.id_users;

        axios.post(base_url + "/ready-send-orders/" + id_sopir + "/" + this.perPage + "/"+offset,
        form, conf)
        .then(response => {
          this.orders = response.data.orders;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          alert(error);
        })
      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/driver/check", form)
        .then(response => {
          if (response.data.auth == false) {
            window.location = "login.html";
          } else{
            this.key = token;
            this.users = response.data.driver;
            this.find();
          }
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
