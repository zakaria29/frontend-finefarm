<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Pengiriman Order
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
            </b-col>
            <b-col cols="5">
              <h4>Customer: {{ item.pembeli.nama }}</h4>
              <h4>Kasir: {{ item.users === null ? "-" : item.users.nama }}</h4>
              <h4>Sopir: {{ item.sopir === null ? "-" : item.sopir.nama }}</h4>
            </b-col>
            <b-col>
              <b-button class="btn btn-block btn-sm btn-dark" v-b-modal.modal_send
              @click="send(item)">
                <span class="fa fa-check"></span> Kirim Order
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

    <b-modal id="modal_send" title="Send Order"
      header-bg-variant="light" size="lg"
      border-variant="dark" hide-footer>

      <b-container fluid v-if="selectedOrder !== null">
        <form v-on:submit.prevent="sendOrder">
          <b-row class="mb-1">
            <b-col cols="3">ID Order</b-col>
            <b-col>{{ selectedOrder.id_orders }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Invoice</b-col>
            <b-col>{{ selectedOrder.invoice }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Customer</b-col>
            <b-col>{{ selectedOrder.pembeli.nama }}</b-col>
          </b-row>
          <b-row class="mb-1">
            <b-col cols="3">Nominal</b-col>
            <b-col>{{ "Rp " + formatNumber(selectedOrder.total_bayar) }}</b-col>
          </b-row>
          List Barang
          <ul class="list-group">
            <li class="list-group-item mb-1" v-for="(item,index) in detail_ambil_stok">
              <b-row class="mb-1">
                <b-col cols="4">{{ item.nama_barang }}</b-col>
                <b-col cols="4">{{ item.jumlah_barang }}</b-col>
                <b-col cols="4">
                  <b-button size="sm" variant="success" @click="AddSupplier(index)">
                    <span class="fa fa-plus"></span> Add Supplier List
                  </b-button>
                </b-col>
              </b-row>
              <b-row class="mb-1" v-for="(it, i) in item.supplier">
                <b-col cols="4">
                  <v-select :options="item.suppliers" @input="BindSupplier(index, i)"
                  :reduce="nama_supplier => nama_supplier.id_supplier"
                  label="nama_supplier" v-model="it.id_supplier">
                  <template #search="{attributes, events}">
                    <input
                      class="vs__search"
                      :required="!it.id_supplier"
                      v-bind="attributes"
                      v-on="events"
                    />
                  </template>
                  </v-select>
                  <strong v-if="it.id_supplier !== null">
                    Stok: {{ it.max }}
                  </strong>
                </b-col>
                <b-col cols="4">
                   <b-form-input type="number"
                   v-model="it.jumlah"
                   min="1"
                   :max="it.max" required>
                   </b-form-input>
                </b-col>
                <b-col cols="4">
                  <b-button size="sm" variant="danger" @click="DropSupplier(index,i)">
                    <span class="fa fa-trash"></span> Remove Supplier
                  </b-button>
                </b-col>
              </b-row>
            </li>
          </ul>

          <button type="submit" class="btn btn-sm btn-info btn-block">
            Send Order
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
        supplier: [],
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
        selectedOrder: null,
        detail_ambil_stok: [],
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

      AddSupplier : function(index){
        this.detail_ambil_stok[index].supplier.push({
          id_supplier: null,
          jumlah: 0,
          max: 0
        });
      },

      DropSupplier : function(index, i){
        if (this.detail_ambil_stok[index].supplier.length > 1) {
          this.detail_ambil_stok[index].supplier.splice(i,1);
        } else {

        }
      },

      BindSupplier : function(index, i){
        let id = this.detail_ambil_stok[index].supplier[i].id_supplier;
        if (id !== "") {
          let idb = this.detail_ambil_stok[index].id_barang;
          let item = this.detail_ambil_stok[index].suppliers.find(it => it.id_supplier === id);
          if(item !== null){
            let data = item.stok_barang.find(it => it.id_barang === idb)
            this.detail_ambil_stok[index].supplier[i].max = data.stok;
          }
        }
      },

      send : function(item){
        this.selectedOrder = item;
        this.detail_ambil_stok = [];
        item.detail_orders.forEach((detail, i) => {
          this.detail_ambil_stok.push({
            id_barang: detail.id_barang,
            nama_barang: detail.barang.nama_barang,
            jumlah_barang: detail.jumlah_barang,
            suppliers: this.supplier.filter(it => {
              if(it.stok_barang.some(sb => sb.id_barang === detail.id_barang))
              return it
            }),
            supplier: [{
              id_supplier: null,
              jumlah: 0,
              max: 0
            }]
          });
        });
      },

      sendOrder : function(){
        if (confirm("Apakah Anda yakin ingin mengirim order ini?")) {
          this.$bvToast.show("loading");
          this.$bvModal.hide("modal_send");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("id_users", this.users.id_users);
          form.append("detail_ambil_stok", JSON.stringify(this.detail_ambil_stok));

          axios.post(base_url + "/send-orders/" + this.selectedOrder.id_orders, form, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.find();
            window.open(base_url + "/struk/" + this.selectedOrder.id_orders,'_blank');
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

        axios.post(base_url + "/prepare-orders/" + this.perPage + "/" + offset, form, conf)
        .then(response => {
          this.orders = response.data.orders;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          alert(error);
        })
      },

      get_supplier : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/supplier", conf)
        .then(response => {
          this.supplier = response.data;
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
            this.get_supplier();
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
