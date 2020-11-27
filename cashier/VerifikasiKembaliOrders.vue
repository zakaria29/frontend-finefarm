<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Verifikasi Pengurangan Order
      </h3>
    </div>
    <div class="card-body">

      <b-alert class="mt-2" variant="warning" :show="kembaliOrders.length === 0">
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

      <div class="mt-2" v-if="kembaliOrders.length > 0">
        <h4>Verifikasi Barang yang dikembalikan</h4>
        <ul class="mb-1 list-group">
          <li class="list-group-item" v-for="item in kembaliOrders">
            <b-row>
              <b-col cols="9">
                <h4>ID Order: {{ item.orders.id_orders }}</h4>
                <h4>Waktu Kirim: {{ formatDate(item.orders.waktu_pengiriman) }}</h4>
                <h4>Customer: {{ item.orders.pembeli.nama }}</h4>
                <h3 class="text-info">Total: Rp {{ formatNumber(item.orders.total_bayar) }}</h3>
              </b-col>
              <b-col>
                <b-button class="btn btn-block btn-sm btn-primary" v-b-modal.modal_verify
                @click="Verify(item)">
                  <span class="fa fa-check"></span> Verifikasi
                </b-button>

                <!-- <b-button class="btn btn-block btn-sm btn-success" v-b-modal.modal_detail
                @click="Detail(item)">
                  <span class="fa fa-eye"></span> Detail Order
                </b-button> -->
              </b-col>
            </b-row>
          </li>
        </ul>
      </div>

      <b-modal id="modal_verify" title="Verifikasi Pengurangan Order"
        header-bg-variant="success" size="lg"
        border-variant="info" hide-footer>
        <form v-on:submit.prevent="VerifyOrders">
          <b-container fluid v-if="selectedOrder !== null">
            <b-row class="mb-3">
              <b-col cols="4">Customer</b-col>
              <b-col>: {{ selectedOrder.orders.pembeli.nama }}</b-col>
            </b-row>

            <b-row class="mb-2">
              <b-col>
                <ul class="list-group">
                  <li class="list-group-item">
                    <b-row>
                      <b-col cols="6">
                        <small class="text-success"><i>Nama Barang</i></small>
                      </b-col>
                      <b-col cols="3">
                        <small class="text-danger"><i>Dikurangi sejumlah</i></small>
                      </b-col>
                      <b-col>
                        <small class="text-warning"> <i>Supplier</i> </small>
                      </b-col>
                    </b-row>
                  </li>
                  <li class="list-group-item" v-for="b in selectedKembaliOrders">
                    <b-row>
                      <b-col cols="6">{{ b.nama_barang }}</b-col>
                      <b-col cols="3">
                        <b-form-input type="number" v-model="b.jumlah_barang"
                         size="sm">
                        </b-form-input>
                      </b-col>
                      <b-col>
                        <v-select :options="suppliers"
                        :reduce="nama_supplier => nama_supplier.id_supplier"
                        label="nama_supplier" v-model="b.id_supplier">
                        <template #search="{attributes, events}">
                          <input
                            class="vs__search"
                            :required="!b.id_supplier"
                            v-bind="attributes"
                            v-on="events"
                          />
                        </template>
                        </v-select>
                      </b-col>
                    </b-row>
                  </li>
                </ul>
              </b-col>
            </b-row>

            <button type="submit" class="btn btn-block btn-dark">
              Verify
            </button>
          </b-container>
        </form>
      </b-modal>

  </div>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        kembaliOrders: [],
        setorUang: [],
        id_sopir: null,
        message : "",
        key: "",
        selectedOrder: null,
        selectedKembaliOrders: [],
        suppliers: [],
      }
    },

    methods: {
      get_kembali_orders : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/kembali-orders")
        .then(response => {
          this.kembaliOrders = response.data;
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
        })
        .catch(error => {
          alert(error);
        })
      },

      Verify : function(item){
        this.selectedKembaliOrders = [];
        this.selectedOrder = item;
        item.detail_kembali_orders.forEach((it, i) => {
          this.selectedKembaliOrders.push({
            id_barang: it.id_barang,
            jumlah_barang: it.jumlah_barang,
            id_supplier: "",
            nama_barang: it.barang.nama_barang
          })
        });
        ;
      },

      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          if(item.status === "1")
          total += Number(item.nominal)
        });
        return total;
      },

      VerifyOrders : function(){
        if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
          this.$bvModal.hide("modal_verify");
          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("id_kembali_orders",this.selectedOrder.id_kembali_orders);
          form.append("id_users",this.selectedOrder.id_sopir);
          form.append("detail_kembali_orders", JSON.stringify(this.selectedKembaliOrders));
          // console.log(this.selectedKembaliOrders);
          axios.post(base_url + "/verify-kembali-orders", form, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.get_kembali_orders();
          })
          .catch(error => {
            console.log(error);
          })
        }
      },
    },

    mounted(){
      this.get_kembali_orders();
    }
  }
</script>
