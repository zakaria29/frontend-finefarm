<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Verifikasi Order
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
            <b-button type="submit" class="btn btn-info btn-block">
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
              <b-row>
                <b-col>
                  <h3 class="text-info">
                    Total: Rp {{ formatNumber(item.total_bayar) }}
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
              <b-button class="btn btn-block btn-sm btn-dark" v-b-modal.modal_accept
              @click="accept(item)">
                <span class="fa fa-check"></span> Verify Order
              </b-button>

              <b-button class="btn btn-block btn-sm btn-info" v-if="(item.id_status_orders < 2)"
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

    <b-modal id="modal_accept" title="Accept Order"
      header-bg-variant="light" size="lg"
      border-variant="dark" hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="acceptOrder">
          <b-row class="mb-1">
            <b-col cols="3">ID Order</b-col>
            <b-col>{{ selected.id_orders }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Invoice</b-col>
            <b-col>{{ selected.invoice }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Customer</b-col>
            <b-col>{{ selected.customer }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Nominal</b-col>
            <b-col>{{ "Rp " + formatNumber(selected.total_bayar) }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Sopir</b-col>
            <b-col>
              <v-select :options="driver"
              :reduce="nama => nama.id_users" label="nama" v-model="selected.id_sopir">
              <template #search="{attributes, events}">
                <input
                  class="vs__search"
                  :required="!selected.id_sopir"
                  v-bind="attributes"
                  v-on="events"
                />
              </template>
              </v-select>
            </b-col>
          </b-row>

          <b-row class="my-2">
            <b-col>
              <ul class="list-group">
                <li class="list-group-item text-bold">
                  <b-row>
                    <b-col cols="4">Items</b-col>
                    <b-col cols="2">Qty</b-col>
                    <b-col cols="3">Harga Satuan</b-col>
                    <b-col cols="3">Total</b-col>
                  </b-row>
                </li>
                <li class="list-group-item" v-for="(item, index) in selected.detail_orders">
                  <b-row>
                    <b-col cols="4">{{ item.barang.nama_barang }}</b-col>
                    <b-col cols="2">
                      {{ item.jumlah_barang }}
                      <small>{{ (item.barang.satuan === "1") ? " Kg" : " Butir" }}</small>
                    </b-col>
                    <b-col cols="3">
                      {{ formatNumber(item.harga_beli) }}
                      <a type="button" class="text-warning"
                      v-if="item.status_harga_order === true" @click="changeHarga(index, true)">
                        <small>Gunakan Harga Terbaru</small>
                      </a>
                      <a type="button" class="text-success"
                      v-else @click="changeHarga(index, false)">
                        <small>Gunakan Harga Order</small>
                      </a>
                    </b-col>
                    <b-col cols="3">
                      {{ formatNumber(item.jumlah_barang * item.harga_beli) }}
                    </b-col>
                  </b-row>
                  <b-row>
                    <b-col cols="4">{{ item.pack.nama_pack }}</b-col>
                    <b-col cols="2">{{ item.jumlah_pack }}</b-col>
                    <b-col cols="3">
                      {{ formatNumber(item.harga_pack) }}
                    </b-col>
                    <b-col cols="3">
                      {{ formatNumber(item.jumlah_pack * item.harga_pack) }}
                    </b-col>
                  </b-row>
                </li>
              </ul>
            </b-col>
          </b-row>
          <button type="submit" class="btn btn-sm btn-info btn-block">
            Accept Order
          </button>
        </form>
      </b-container>
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
        driver: [],
        orders: [],
        detail_orders: [],
        log_orders: [],
        totalRow: 0,
        perPage: 10,
        currentPage: 1,
        from: "",
        to: "",
        search: "",
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
        selected: {
          id_sopir: null,
          po: null,
          invoice: null,
          total_bayar: null,
          customer: null,
          id_orders: null,
          detail_orders: []
        }
      }
    },

    methods: {
      changeHarga : function(index, status){
        if(status){
          let harga = this.selected.detail_orders[index].current_harga;
          this.selected.detail_orders[index].harga_beli = harga;
          this.selected.detail_orders[index].status_harga_order = false;
          if (this.selected.detail_orders[index].original_is_lock) {
            this.selected.detail_orders[index].is_lock = false;
          }
        }
        else{
          let harga = this.selected.detail_orders[index].harga_order;
          this.selected.detail_orders[index].harga_beli = harga;
          this.selected.detail_orders[index].status_harga_order = true;
          if (this.selected.detail_orders[index].original_is_lock) {
            this.selected.detail_orders[index].is_lock = true;
          }
        }
      },
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

      accept : function(item){
        this.selected.detail_orders = [];
        this.selected.id_orders = item.id_orders;
        this.selected.total_bayar = item.total_bayar;
        this.selected.invoice = item.invoice;
        this.selected.po = item.po;
        this.selected.customer = item.pembeli.nama;
        this.selected.id_sopir = null;
        item.detail_orders.forEach((it, i) => {
          this.selected.detail_orders.push({
            id_orders: item.id_orders,
            id_barang: it.id_barang,
            id_pack: it.id_pack,
            jumlah_barang: it.jumlah_barang,
            jumlah_pack: it.jumlah_pack,
            harga_pack: it.harga_pack,
            harga_beli: it.harga_beli,
            original_is_lock: it.is_lock,
            is_lock: it.is_lock,
            harga_order: it.harga_beli,
            current_harga: Number(item.pembeli.margin) + Number(it.barang.current_harga.harga) +
            Number(item.pembeli.group_customer.margin),
            status_harga_order: true,
            pack: it.pack,
            barang: it.barang
          })
        });
      },

      acceptOrder : function(){
        if (confirm("Apakah Anda yakin akan verifikasi order ini?")) {
          this.$bvToast.show("loading");
          this.$bvModal.hide("modal_accept");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("id_users", this.users.id_users);
          form.append("id_sopir", this.selected.id_sopir);
          form.append("detail_orders", JSON.stringify(this.selected.detail_orders));

          axios.post(base_url + "/accept-orders/" + this.selected.id_orders, form, conf)
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

      find : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("find", this.search);
        let offset = (this.currentPage-1) * this.perPage;

        axios.post(base_url + "/verify-orders/" + this.perPage + "/" + offset, form, conf)
        .then(response => {
          this.orders = response.data.orders;
          this.totalRow = response.data.count;

          this.get_driver();
        })
        .catch(error => {
          alert(error);
        })
      },

      get_driver : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/driver", conf)
        .then(response => {
          this.driver = response.data.driver;
        })
        .catch(error => {
          alert(error);
        })
      },

      initUser: function(){
        let token = this.$cookies.get("Api-Token");
        let form = new FormData();
        form.append("token",token);
        axios.post(base_url+"/cashier/check", form)
        .then(response => {
          if (response.data.auth == false) {
            window.location = "login.html";
          } else{
            this.key = token;
            this.users = response.data.cashier;
            this.orders.id_users = response.data.cashier.id_users;
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
