<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Verifikasi Retur Order
      </h3>
    </div>
    <div class="card-body">
      <b-alert class="mt-2" variant="warning" :show="returOrder.length === 0">
        Tidak Ada Data yang perlu diverifikasi
      </b-alert>

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <div class="mt-2" v-if="returOrder.length > 0">
        <h4>Verifikasi Barang yang dikembalikan</h4>
        <ul class="mb-1 list-group">
          <li class="list-group-item" v-for="(item, index) in returOrder">
            <h4>Invoice: {{ item.invoice }}</h4>
            <h4>Waktu Kirim: {{ formatDate(item.waktu_pengiriman) }}</h4>
            <h4>Customer: {{ item.pembeli.nama }}</h4>
            <strong>Detail Retur</strong>

            <b-button size="sm" class="btn m-2 btn-sm btn-warning" v-b-modal.modal_detail
            @click="Detail(item)">
              <span class="fa fa-history"></span> List Order Awal
            </b-button>

            <form v-on:submit.prevent="VerifyRetur(index)">
              <div class="input-group mb-1">
                <div class="input-group-prepend">
                  <div class="input-group-text">Tgl. Pengiriman</div>
                </div>
                <input type="date" v-model="item.waktu_pengiriman" class="form-control" required />
              </div>
              <ul class="list-group">
                <li class="mb-1 list-group-item" v-for="detail in item.detail_retur_order">
                  <b-row>
                    <b-col cols="4">
                      <small class="text-info"> <strong>Nama Barang</strong> </small> <br />
                      {{ detail.barang.nama_barang }}
                    </b-col>
                    <b-col cols="2">
                      <small class="text-info">
                        <strong>
                          Jml. Retur Barang
                          ({{ detail.barang.satuan === "1" ? "Kg" : "butir" }})
                        </strong>
                      </small>
                      <input type="number" v-model="detail.jumlah_barang"
                      class="form-control form-control-sm" />

                    </b-col>
                    <b-col cols="2">
                      <small class="text-info"><strong>Jml. Retur Pack</strong> </small>
                      <input type="number" v-model="detail.jumlah_pack"
                      class="form-control form-control-sm" />
                    </b-col>
                    <b-col cols="4">
                      <small class="text-info"> <strong>Supplier</strong> </small>
                      <select class="form-control form-control-sm"
                      v-model="detail.id_supplier" required>
                        <option v-for="s in suppliers" :value="s.id_supplier">
                          {{ s.nama_supplier }}
                        </option>
                      </select>
                    </b-col>
                  </b-row>
                  <b-row>
                    <b-col cols="4">
                      Pengiriman Ulang
                    </b-col>
                    <b-col>
                      <small class="text-info"> <strong>Jenis Pack</strong> </small>
                      <select class="form-control form-control-sm"
                      required v-model="detail.id_pack_new">
                        <option v-for="p in packs" :value="p.id_pack">{{ p.nama_pack }}</option>
                      </select>
                    </b-col>
                    <b-col>
                      <small class="text-info"> <strong>Jumlah Pack</strong> </small>
                      <input type="number" class="form-control form-control-sm"
                      v-model="detail.jumlah_pack_new" required />
                    </b-col>
                  </b-row>
                </li>
              </ul>
              <b-button type="submit" class="btn btn-block btn-sm btn-success">
                Verify
              </b-button>
            </form>
          </li>
        </ul>
      </div>

      <!-- modall detail -->
      <b-modal id="modal_detail" title="Order Awal"
        header-bg-variant="danger" size="lg"
        border-variant="info" hide-footer>

        <b-container fluid>
          <!-- <div class="row mb-3">
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
          </div> -->
          <div class="row mb-3">
            <ul class="list-group col-12">
              <li class="list-group-item mb-2" v-for="d in detail_orders">
                <div class="row mb-2">
                  <div class="col-3">
                    <strong>{{ d.barang.nama_barang }}</strong>
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
                <strong>Pack</strong>
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
              <!-- <li class="list-group-item mb-2">
                <div class="row">
                  <div class="col-8">

                  </div>
                  <div class="col-4">
                    {{ "Rp " + formatNumber(order.total_bayar) }}
                  </div>
                </div>
              </li> -->
            </ul>
          </div>
        </b-container>
      </b-modal>
    </div>
  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        message : "",
        key: "",
        returOrder: [],
        suppliers: [],
        packs: [],
        selectedReturOrder: null,
        detail_orders: [],
      }
    },

    methods: {
      get_retur_order : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/retur-order")
        .then(response => {
          let retur = response.data;
          for (var i = 0; i < retur.length; i++) {
            for (var j = 0; j < retur[i].detail_retur_order.length; j++) {
              retur[i].detail_retur_order[j].id_supplier = "";
              retur[i].detail_retur_order[j].id_pack_new = "";
              retur[i].detail_retur_order[j].jumlah_pack_new = "";
            }
          }
          this.returOrder = retur;
          this.get_supplier();
        })
        .catch(error => {
          alert(error);
        })
      },

      get_supplier : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/supplier", conf)
        .then(response => {
          this.suppliers = response.data;
          this.get_pack();
        })
        .catch(error => {
          alert(error);
        })
      },

      get_pack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/pack", conf)
        .then(response => {
          this.packs = response.data.pack;
        })
        .catch(error => {
          alert(error);
        })
      },

      Detail : function(item){
        this.detail_orders = item.detail_orders
      },

      VerifyRetur : function(index){
        if (confirm("Apakah Anda yakin dengan data retur ini?")) {
          let conf = { headers: { "Api-Token" : this.key} };
          this.$bvToast.show("loading");
          let selectedRetur = this.returOrder[index];

          let form = new FormData();
          form.append("id_retur_order", selectedRetur.id_retur_order)
          form.append("detail_retur_order", JSON.stringify(selectedRetur.detail_retur_order))
          form.append("id_sopir", selectedRetur.id_sopir)
          form.append("waktu_pengiriman", selectedRetur.waktu_pengiriman)
          form.append("id_users", this.users.id_users)
          form.append("id_pembeli", selectedRetur.id_pembeli)
          axios.post(base_url + "/retur-order", form,conf)
          .then(response => {
            console.log(response.data);
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.get_retur_order();
          })
          .catch(error => {
            alert(error);
          })
        }
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
            this.get_retur_order();
          }
        })
        .catch(error => {
          console.log(error);
        });
      },
    },

    mounted(){
      this.initUser()
    }
  }
</script>
