<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="acquisitions-incorporated"></span> Order
      </h3>
    </div>

    <div class="card-body">
      <form v-on:submit.prevent="Save">
        <div class="row mb-2">
          <div class="col-3">
            Customer
          </div>
          <div class="col-9">
            <v-select :options="customer" @input="Customer()"
            :reduce="nama => nama.id_users" label="nama" v-model="orders.id_pembeli">
            <template #search="{attributes, events}">
              <input
                class="vs__search"
                :required="!orders.id_pembeli"
                v-bind="attributes"
                v-on="events"
              />
            </template>
            </v-select>
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-sm-3">
            Waktu Order
          </div>
          <div class="col-sm-5">
            <input type="date" v-model="orders.date_order" class="form-control" required />
          </div>
          <div class="col-sm-4">
            <input type="time" v-model="orders.time_order" class="form-control" required />
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-3">
            Waktu Pengiriman
          </div>
          <div class="col-3">
            <input type="date" v-model="orders.waktu_pengiriman" @change="setJatuhTempo()"
            class="form-control" required />
          </div>
          <div class="col-2">
            Jatuh Tempo
          </div>
          <div class="col-4">
            <input type="date" v-model="orders.tgl_jatuh_tempo"
            class="form-control" required />
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-3">
            PO
          </div>
          <div class="col-3">
            <input type="text" v-model="orders.po"
            class="form-control" required />
          </div>
          <div class="col-2">
            Invoice
          </div>
          <div class="col-4">
            <input type="text" v-model="orders.invoice"
            class="form-control" required />
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-3">
            Tipe Pembayaran
          </div>
          <div class="col-3">
            <b-form-select v-model="orders.tipe_pembayaran" required @change="Tipe">
              <option value="0">Kredit</option>
              <option value="1">Tunai</option>
              <option value="2">Down Payment</option>
            </b-form-select>
          </div>
          <div class="col-2" v-if="dp">
            Down Payment
          </div>
          <div class="col-4" v-if="dp">
            <input type="number" v-model="orders.down_payment"
            class="form-control" required />
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-3">
            Sopir
          </div>
          <div class="col-6">
            <v-select :options="driver"
            :reduce="nama => nama.id_users" label="nama" v-model="orders.id_sopir">
            <template #search="{attributes, events}">
              <input
                class="vs__search"
                :required="!orders.id_sopir"
                v-bind="attributes"
                v-on="events"
              />
            </template>
            </v-select>
          </div>
        </div>

        <div class="row col-12 mb-2">
          <h4>List Barang</h4>
          <br />
          <ul class="list-group col-12">
            <li v-for="(d,index) in detail_order" class="list-group-item mb-2">
              <div class="row mb-2">
                <div class="col-3">
                  Pilih Barang
                  <b-form-select v-model="d.id_barang" required
                  @change="SelectBarang(d.id_barang,index)">
                    <option v-for="b in barang" :value="b.id_barang">
                      {{ b.nama_barang }}
                    </option>
                  </b-form-select>
                </div>

                <div class="col-2">
                  Qty
                   <b-form-input v-model="d.jumlah_barang" type="number"
                   v-on:keyup="SelectPack(index)" @change="CountTotal()">
                 </b-form-input>
                </div>

                <div class="col-3">
                  Harga {{ d.is_lock ? "Lock" : "Saat Ini" }}
                  <br />
                  {{ "Rp " + formatNumber(d.harga_beli) }}
                 <b-button block size="sm" :variant="d.is_lock ? 'primary' : 'success'"
                  v-if="users_lock"
                 @click="() => {d.is_lock = !d.is_lock; SelectBarang(d.id_barang, index)}">
                   {{ d.is_lock ? "Pakai Harga Saat ini" : "Pakai Harga Lock" }}
                 </b-button>
                 <b-button block size="sm" variant="dark" v-else
                 @click="SelectBarang(d.id_barang, index)">
                   Set Harga Saat ini
                 </b-button>
                </div>

                <div class="col-4">
                  Total <br />
                  {{ "Rp " +
                  formatNumber(Number(d.jumlah_barang * d.harga_beli) +
                  Number(d.jumlah_pack * d.harga_pack)) }}
                </div>
              </div>
              <div class="row mb-2">
                <div class="col-3">
                  Pilih Pack
                  <b-form-select v-model="d.id_pack" required
                  @change="SelectPack(index)">
                    <option v-for="p in d.pack" :value="p.id_pack">
                      {{ p.nama_pack }}
                    </option>
                  </b-form-select>
                </div>
                <div class="col-2">
                  Jumlah Pack
                  <b-form-input v-model="d.jumlah_pack" type="number" min="1" required>
                  </b-form-input>
                </div>

                <div class="col-2">
                  Keterangan Pack
                  <br />
                   <b-form-checkbox v-model="d.harga_pack" switch @click="BeliPack(index)"
                   :value="d.beli_pack" unchecked-value="">
                    {{ (d.harga_pack > 0) ? "Beli Pack" : "Pinjam Pack" }}
                  </b-form-checkbox>
                </div>
              </div>
              <div class="row mb-2">
                <b-button class="btn btn-sm btn-danger btn-block" @click="Drop(index)">
                  <span class="fa fa-trash"></span> Hapus
                </b-button>
              </div>
            </li>
          </ul>
        </div>

        <div class="row mb-2">
          <div class="col-8">
            <h2>Total</h2>
          </div>
          <div class="col-4">
            <!-- <h2>Rp {{ formatNumber(CountTotal()) }}</h2>
            <br /> -->
            <input type="number" class="form-control"
            v-model="orders.total_bayar" />
            <!-- {{ orders.total_bayar }} -->
          </div>

        </div>
        <div class="row mb-2">
          <b-button variant="info" @click="Add">
            <span class="fa fa-plus"></span> Tambah List
          </b-button>
        </div>

        <div class="row mb-2">
          <b-form-textarea v-model="orders.catatan" placeholder="Tambahkan Catatan Order" rows="3"
          max-rows="6">
          </b-form-textarea>
        </div>

        <div class="row mb-2">
          <b-button class="btn btn-block btn-default" type="submit">
            <span class="fa fa-check"></span> Order
          </b-button>
        </div>
      </form>
    </div>

    <!-- toast -->
    <!-- toast untuk tampilan message box -->
    <b-toast id="message" title="Message" solid>
      <strong class="text-success">{{ message }}</strong>
    </b-toast>

    <b-toast id="loading" title="Processing Data" solid no-auto-hide>
      <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
      <span class="fa fa-spinner fa-spin"></span>
      <strong class="text-success">Loading...</strong>
    </b-toast>

  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        key: "",
        message: "",
        margin: 0,
        margin_group: 0,
        driver: [],
        barang: [],
        tempBarang: [],
        customer: [],
        detail_order: [],
        orders: {
          id_orders: "",
          id_users: "",
          id_pembeli: "",
          id_sopir: "",
          id_status_orders: "",
          date_order: "",
          time_order: "",
          waktu_pengiriman: "",
          tgl_jatuh_tempo: "",
          tipe_pembayaran: "",
          total_bayar: 0,
          down_payment: 0,
          po: "",
          invoice: "",
          catatan: "",
        },
        dp: false,
        users_lock : false,
      }
    },

    methods: {
      Drop : function(index){
        this.detail_order.splice(index,1);
      },

      Add : function(){
        this.detail_order.push({
          id_barang: "",
          pack: [],
          id_pack: "",
          jumlah_barang: 0,
          harga_beli: 0,
          harga_pack: "",
          jumlah_pack: 0,
          beli_pack : 0,
          is_lock: this.users_lock,
        });
      },

      SelectBarang : function(id,index){
        let item = this.barang.find(it => it.id_barang === id);
        this.detail_order[index].pack = item.pack;
        this.detail_order[index].satuan = item.satuan === "1" ? "Kg" : "Butir";

        this.detail_order[index].harga_beli =
        Number(this.margin) + Number(item.harga) + Number(this.margin_group);
        if (this.users_lock) {
          if (this.detail_order[index].is_lock) {
            this.detail_order[index].harga_beli = Number(item.harga_lock);
          }
        }
        this.detail_order[index].id_pack = item.pack[0].id_pack;
        this.CountTotal();
      },

      SelectPack : function(index){
        let id = this.detail_order[index].id_pack;
        let item = this.detail_order[index].pack.find(it => it.id_pack === id);
        let barang = this.barang.find(it => it.id_barang === this.detail_order[index].id_barang);
        if (barang.satuan === "1") {
          this.detail_order[index].jumlah_pack =
          Math.ceil(Number(this.detail_order[index].jumlah_barang) * Number(item.kapasitas_kg));
        }else if (barang.satuan === "2") {
          this.detail_order[index].jumlah_pack =
          Math.ceil(Number(this.detail_order[index].jumlah_barang) * Number(item.kapasitas_butir));
        }
        this.detail_order[index].beli_pack = item.harga;
        this.BeliPack(index);
        this.CountTotal();
      },

      BeliPack : function(index){
        if (this.detail_order[index].harga_pack > 0) {
          let id_pack = this.detail_order[index].id_pack;
          let pack = this.detail_order[index].pack.find(it => it.id_pack === id_pack);
          this.detail_order[index].harga_pack = pack.harga;
        }else{
          this.detail_order[index].harga_pack = 0;
        }
        this.CountTotal();
      },

      Customer : function(){
        let item = this.customer.find(it => it.id_users === this.orders.id_pembeli);
        this.margin = item.margin;
        this.margin_group = item.margin_group;
        // this.detail_order = [];

        if(item.lock_pack_barang.length > 0){
          this.barang = [];
          let brg = null;
          item.lock_pack_barang.forEach((it, i) => {
            brg = this.tempBarang.find(itm => itm.id_barang === it.id_barang);
            brg.harga_lock = it.harga;
            this.barang.push(brg);
          });
          this.users_lock = true;
        }else{
          this.barang = this.tempBarang;
          this.users_lock = false;
        }
      },

      CountTotal : function(){
        let total = 0;
        this.detail_order.map(item => {
          total += (Number(item.harga_beli) * Number(item.jumlah_barang)) +
          (Number(item.harga_pack) * Number(item.jumlah_pack));
        });
        this.orders.total_bayar = total;
        // return total;
      },

      Tipe : function(){
        if (this.orders.tipe_pembayaran == "2") {
          this.dp = true;
        }else{
          this.dp = false;
        }
        this.orders.down_payment = 0;
      },
      get_customer : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/customer", conf)
        .then(response => {
          this.customer = response.data.customer;
          this.get_driver();
        })
        .catch(error => {
          alert(error);
        })
      },

      get_barang : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/barang", conf)
        .then(response => {
          this.barang = response.data.barang;
          this.tempBarang = response.data.barang;
          this.get_customer();
        })
        .catch(error => {
          alert(error);
        })
      },

      Save : function(){
        if (this.detail_order.length > 0) {
          if (confirm("Apakah Anda yakin dengan order ini?")) {
            let conf = { headers: { "Api-Token" : this.key} };
            this.$bvToast.show("loading");
            let form = new FormData();
            form.append("id_orders", this.orders.id_orders);
            form.append("id_users", this.orders.id_users); // sementara
            form.append("id_pembeli", this.orders.id_pembeli);
            form.append("id_sopir", this.orders.id_sopir);
            form.append("waktu_order",this.orders.date_order + " " + this.orders.time_order);
            form.append("waktu_pengiriman", this.orders.waktu_pengiriman);
            form.append("tgl_jatuh_tempo", this.orders.tgl_jatuh_tempo);
            form.append("tipe_pembayaran", this.orders.tipe_pembayaran);
            form.append("total_bayar", this.orders.total_bayar);
            form.append("down_payment", this.orders.down_payment);
            form.append("po", this.orders.po);
            form.append("invoice", this.orders.invoice);
            form.append("catatan", this.orders.catatan);
            form.append("verify", this.orders.id_status_orders === '1' ? true : false);
            form.append("detail_orders", JSON.stringify(this.detail_order));

            axios.post(base_url + "/orders/update_order", form, conf)
            .then(response => {
              this.$bvToast.hide("loading");
              this.message = response.data.message;
              this.$bvToast.show("message");
              this.get_order();
            })
            .catch(error => {
              alert(error);
            })
          }
        }else{
          alert("Anda belum memilih list barang");
        }
      },

      get_driver : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/driver", conf)
        .then(response => {
          this.driver = response.data.driver;
          this.Customer();
        })
        .catch(error => {
          alert(error);
        })
      },

      setJatuhTempo : function(){
        let send = new Date(this.orders.waktu_pengiriman)
        // console.log(this.jatuh_tempo);
        send.setDate(Number(send.getDate()) + Number(this.jatuh_tempo))
        this.orders.tgl_jatuh_tempo = send.toISOString().slice(0,10);;
        // console.log(this.orders.tgl_jatuh_tempo);
      },

      get_order : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/orders/get/" + this.orders.id_orders)
        .then(response => {
          console.log(response.data);
          this.margin =  response.data.orders.margin;
          this.margin_group = response.data.orders.margin_group;
          this.detail_order = response.data.orders.detail_orders;
          // this.orders.id_users =  response.data.orders.id_users === null ?
          // null : this.users.id_users;
          this.orders.id_pembeli =  response.data.orders.id_pembeli;
          this.orders.id_sopir =  response.data.orders.id_sopir;
          this.orders.id_status_orders = response.data.orders.id_status_orders;

          this.orders.waktu_pengiriman =  response.data.orders.waktu_pengiriman;
          this.orders.tgl_jatuh_tempo =  response.data.orders.tgl_jatuh_tempo;
          this.orders.tipe_pembayaran =
          response.data.orders.down_payment > 0 ? "2" : response.data.orders.tipe_pembayaran;
          this.orders.total_bayar =  response.data.orders.total_bayar;
          this.orders.down_payment =  response.data.orders.down_payment;
          this.orders.po =  response.data.orders.po;
          this.orders.invoice =  response.data.orders.invoice;
          this.orders.catatan =  response.data.orders.catatan;
          this.dp= response.data.orders.down_payment > 0 ? true : false;

          let date = new Date(response.data.orders.waktu_order);
          let currentDate = date.toISOString().slice(0,10);
          let currentTime = date.getHours() + ':' + date.getMinutes();
          this.orders.date_order = currentDate;
          // this.supply.time = currentTime;
          this.orders.time_order = (date.getHours() < 10 ? "0"+date.getHours() : date.getHours())
          + ":" + (date.getMinutes() < 10 ? "0"+date.getMinutes() : date.getMinutes());

          this.get_barang();
        })
        .catch(error => {
          alert(error);
        });
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
            this.get_order();
          }
        })
        .catch(error => {
          console.log(error);
        });
      },
    },


    mounted(){
      this.orders.id_orders = this.$route.params.id_orders;
      this.initUser();
    }
  }
</script>
