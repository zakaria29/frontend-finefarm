<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Penerimaan Order
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
            </b-col>
            <b-col cols="5">
              <h4>Customer: {{ item.pembeli.nama }}</h4>
              <h4>Kasir: {{ item.users === null ? "-" : item.users.nama }}</h4>
              <h4>Sopir: {{ item.sopir === null ? "-" : item.sopir.nama }}</h4>
            </b-col>
            <b-col>
              <b-button class="btn btn-block btn-sm btn-info" v-b-modal.modal_delivered
              @click="DeliverOrder(item)">
                <span class="fa fa-check"></span> Order Diterima
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
          <b-row>
            <b-col>
              <h3 class="text-info">Total: Rp {{ formatNumber(item.total_bayar) }}</h3>
            </b-col>
            <b-col cols="7">
              <b-badge pill :variant="type[item.id_status_orders]">
                Status: {{ item.status_orders.nama_status_order }}
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

    <b-modal id="modal_delivered" title="Penerimaan Order"
      header-bg-variant="success" size="lg"
      border-variant="info" hide-footer>
      <form v-on:submit.prevent="DeliveredOrder">
        <b-container fluid v-if="selectedOrder !== null">
          <b-row class="mb-1">
            <b-col cols="4">Customer</b-col>
            <b-col>: {{ selectedOrder.pembeli.nama }}</b-col>
          </b-row>

          <b-row class="mb-1">
            <b-col cols="4">Tanggungan Pembayaran</b-col>
            <b-col>: {{ "Rp " + formatNumber(getTotal(selectedOrder.bill)) }}</b-col>
          </b-row>

          <b-row class="mb-1">
            <b-col cols="4">Tanggungan Pack</b-col>
            <b-col>
              <ul>
                <li v-for="p in selectedOrder.tanggungan_pack">
                  <b-row v-if="p.jumlah > 0">
                    <b-col cols="6">{{ p.pack.nama_pack }}</b-col>
                    <b-col>: {{ p.jumlah }}</b-col>
                  </b-row>
                </li>
              </ul>
            </b-col>
          </b-row>

          <b-row class="mb-1">
            <b-col cols="5">
              <b-form-checkbox v-model="statusSetorUang"
              @change="toggleUang(selectedOrder.id_orders)">
               Setor Uang
             </b-form-checkbox>
            </b-col>
            <b-col v-if="setorUang.length > 0">
              <strong>{{ "Total Setor: Rp " + formatNumber(getTotalSetor(setorUang)) }}</strong>
            </b-col>
          </b-row>


          <b-row class="mb-1" v-if="statusSetorUang">
            <b-col>
              <b-form-checkbox-group :options="selectedOrder.bill" value-field="id_orders"
               text-field="label" v-model="setorUang">
             </b-form-checkbox-group>
            </b-col>
          </b-row>

          <b-row class="mb-1">
            <b-col>
              <b-form-checkbox v-model="statusKembaliPack" @change="togglePack">
               Setor Pack
             </b-form-checkbox>
            </b-col>
          </b-row>

          <b-row class="mb-1" v-if="kembaliPack.length > 0">
            <b-col>
              <ul class="list-group">
                <li class="list-group-item" v-for="kp in kembaliPack">
                  <b-row v-if="kp.max > 0">
                    <b-col cols="6">{{ kp.nama_pack }}</b-col>
                    <b-col cols="3">
                       <b-form-input type="number" v-model="kp.jumlah" :max="kp.max"
                        size="sm">
                       </b-form-input>
                    </b-col>
                  </b-row>
                </li>
              </ul>
            </b-col>
          </b-row>
          <button type="submit" class="btn btn-block btn-dark">
            Order Telah Diterima
          </button>
        </b-container>
      </form>
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
        selectedOrder: null,
        setorUang: [],
        kembaliPack: [],
        statusSetorUang: false,
        statusKembaliPack: false,
      }
    },

    methods: {
      Edit : function(id){
        this.$router.push({ name: "EditOrder", params: { id_orders: id }});
      },

      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          total += Number(item.nominal)
        });
        return total;
      },

      getTotalSetor : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          let it = this.selectedOrder.bill.find(b => b.id_orders === item);
          total += Number(it.nominal)
        });
        return total;
      },

      DeliverOrder : function(item){
        this.selectedOrder = item;
        this.setorUang = [];
        this.kembaliPack = [];
        this.statusSetorUang = false;
        this.statusKembaliPack = false;
      },

      togglePack : function(){
        this.statusKembaliPack = !this.statusKembaliPack
        if (this.statusKembaliPack) {
          this.selectedOrder.tanggungan_pack.forEach((item, i) => {
            this.kembaliPack.push({
              id_pack: item.id_pack,
              nama_pack: item.pack.nama_pack,
              jumlah: 0,
              max: item.jumlah
            })
          });

        }else{
          this.kembaliPack = [];
        }
      },

      toggleUang : function(id_orders){
        this.statusSetorUang = !this.statusSetorUang
        this.setorUang = [];
        let index = this.orders.findIndex(it => it.id_orders === id_orders);
        this.orders[index].bill.forEach((item, i) => {
          item.label = "Bill " + (Number(i) + 1) + ": Rp " + formatNumber(item.nominal);
        });

      },

      DeliveredOrder: function(){
        if (confirm("Apakah Anda yakin order ini telah diterima?")) {
          this.$bvToast.show("loading");
          this.$bvModal.hide("modal_delivered");
          let conf = { headers: { "Api-Token" : this.key} };
          let id_orders = this.selectedOrder.id_orders;
          let form = new FormData();
          form.append("id_users", this.users.id_users);
          form.append("id_pembeli", this.selectedOrder.id_pembeli);
          form.append("setor_uang", JSON.stringify(this.setorUang));
          form.append("kembali_pack", JSON.stringify(this.kembaliPack));
          axios.post(base_url + "/delivered-orders/" + id_orders, form, conf)
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

        axios.post(base_url + "/coming-orders/" + id_sopir + "/" + this.perPage + "/"+offset,
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
