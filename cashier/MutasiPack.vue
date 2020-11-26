<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Mutasi Pack
      </h3>
    </div>
    <div class="card-body">
      <form v-on:submit.prevent="getMutasi">
        <b-row>
          <b-col cols="8">
            Pilih Pack
            <b-form-select v-model="id_pack" class="mb-2" required>
              <option v-for="p in pack" :value="p.id_pack">{{ p.nama_pack }}</option>
            </b-form-select>
            Pilih Customer
            <v-select :options="customer"
            :reduce="nama => nama.id_users" label="nama" v-model="id_pembeli">
            <template #search="{attributes, events}">
              <input
                class="vs__search"
                :required="!id_pembeli"
                v-bind="attributes"
                v-on="events"
              />
            </template>
            </v-select>
          </b-col>

          <b-col>
            Tgl. Awal
            <b-form-input class="mb-1" type="date" v-model="from" required></b-form-input>
            Tgl. Akhir
            <b-form-input class="mb-1" type="date" v-model="to" required></b-form-input>
          </b-col>
          <b-button type="submit" block size="sm" variant="success">
            Get Mutasi
          </b-button>
        </b-row>
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
          <template v-slot:cell(beli)="data">
            {{ (data.item.beli > 0) ? data.item.beli : "-" }}
          </template>
          <template v-slot:cell(total_beli)="data">
            {{ data.item.beli > 0 ? "Rp " + formatNumber(data.item.harga * data.item.beli) : "-" }}
          </template>
          <template v-slot:cell(stok)="data">
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
        fields:["waktu","masuk","keluar","beli","total_beli","stok"],
        message : "",
        pack: [],
        id_pack: "",
        customer: [],
        id_pembeli: "",
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
        if (this.id_pembeli !== "0") {
          form.append("id_pembeli", this.id_pembeli);
        }

        axios.post(base_url + "/mutasi-pack/" + this.id_pack, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.mutasi = response.data.log_pack;
        })
        .catch(error => {
          console.log(error);
        })
      },

      get_pack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/pack", conf)
        .then(response => {
          this.pack = response.data.pack;
        })
        .catch(error => {
          console.log(error);
        })
      },

      get_customer : function(){
        let first = [{id_users: "0", nama: "Semua Customer"}];
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/customer", conf)
        .then(response => {
          this.customer = [...first,...response.data.customer];
        })
        .catch(error => {
          console.log(error);
        })
      },

      init : async function(){
        await this.get_pack();
        await this.get_customer();
      },
    },

    mounted(){
      this.init();
    }
  }
</script>
