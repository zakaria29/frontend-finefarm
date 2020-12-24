<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Mutasi Barang
      </h3>
    </div>
    <div class="card-body">
      <form v-on:submit.prevent="getMutasi">
        <b-row class="mb-2">
          <b-col>
            Pilih Barang
            <b-form-select v-model="id_barang" class="mb-2" required size="sm">
              <option v-for="b in barang" :value="b.id_barang">{{ b.nama_barang }}</option>
            </b-form-select>
          </b-col>
          <b-col>
            Pilih Supplier
            <v-select :options="supplier"
            :reduce="nama_supplier => nama_supplier.id_supplier"
            label="nama_supplier" v-model="id_supplier">
            <template #search="{attributes, events}">
              <input
                class="vs__search"
                :required="!id_supplier"
                v-bind="attributes"
                v-on="events"
              />
            </template>
            </v-select>
          </b-col>
          <b-col>
            Tgl. Awal
            <b-form-input class="mb-1" type="date" v-model="from" size="sm" required>
            </b-form-input>
          </b-col>
          <b-col>
            Tgl. Akhir
            <b-form-input class="mb-1" type="date" v-model="to" size="sm" required>
            </b-form-input>
          </b-col>
        </b-row>
        <b-button type="submit" block variant="primary">
          Get Mutasi
        </b-button>
      </form>


      <b-alert class="mt-2" variant="warning" :show="mutasi.length === 0">
        Tidak Ada Data Mutasi
      </b-alert>

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <div class="mt-2" v-if="mutasi.length > 0">
        <b-button variant="success" @click="exportsExcel" class="mb-2">
          <span class="fa fa-file-excel"></span> Export to Excel
        </b-button>
        <b-table :items="mutasi" :fields="fields">
          <template v-slot:cell(waktu)="data">
            {{ formatDate(data.item.waktu) }}
          </template>
          <template v-slot:cell(masuk)="data">
            <strong class="text-success">
              {{ (data.item.masuk > 0) ? data.item.masuk : "-" }}
            </strong>
          </template>
          <template v-slot:cell(keluar)="data">
            <strong class="text-danger">
              {{ (data.item.keluar > 0) ? data.item.keluar : "-" }}
            </strong>
          </template>
          <template v-slot:cell(loss)="data">
            {{ (data.item.loss > 0 || data.item.loss === null) ? data.item.loss : "-" }}
          </template>
          <template v-slot:cell(stok_akhir)="data">
            <strong class="text-info">
              {{ Number(data.item.stok)-Number(data.item.keluar)+Number(data.item.masuk) }}
            </strong>
          </template>
        </b-table>
      </div>
  </div>

</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        mutasi: [],
        fields:["waktu","masuk","keluar","loss","stok_akhir"],
        message : "",
        barang: [],
        supplier: [],
        id_barang: "",
        id_supplier: "",
        from: "",
        to: "",
        key: "",
      }
    },

    methods: {
      getMutasi : function(){
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("from", this.from);
        form.append("to", this.to);
        form.append("id_supplier", this.id_supplier);

        axios.post(base_url + "/mutasi-stok/" + this.id_barang, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.mutasi = response.data.log_stok_barang;
        })
        .catch(error => {
          console.log(error);
        })
      },

      exportsExcel : function(){
        window.open(
          base_url + "/export-mutasi-stok/" + this.id_barang + "/" + this.id_supplier + "/" +
          this.from + "/" + this.to,
          '_blank');
      },

      get_barang : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/barang", conf)
        .then(response => {
          this.barang = response.data.barang;
          this.get_supplier();
        })
        .catch(error => {
          console.log(error);
        })
      },

      get_supplier : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/supplier", conf)
        .then(response => {
          this.supplier = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      init : async function(){
        await this.get_barang();

      },
    },

    mounted(){
      this.init();
    }
  }
</script>
