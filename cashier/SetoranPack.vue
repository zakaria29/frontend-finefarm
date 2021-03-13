<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Setoran Pack
      </h3>
    </div>
    <div class="card-body">
      Pilih Driver (Sopir)
      <v-select :options="driver" @input="getSetoranPack()"
      :reduce="nama => nama.id_users" label="nama" v-model="id_sopir">
      <template #search="{attributes, events}">
        <input
          class="vs__search"
          :required="!id_sopir"
          v-bind="attributes"
          v-on="events"
        />
      </template>
      </v-select>

      <b-alert class="mt-2" variant="warning" :show="id_sopir && setorPack.length === 0">
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

      <div class="mt-2 accordion" role="tablist" v-if="id_sopir && setorPack.length > 0">
        <div class="card mb-1" v-for="p in pack" role="tab">
          <div class="card-header" style="margin:0; padding:0;">
            <b-button variant="info" class="text-left" block style="border-radius:6px"
            v-b-toggle="'acc-' + p.id_pack">
              {{ p.nama_pack }}
              <b-badge pill class="bg-primary text-white mx-2">
                <h4 class="text-white mx-1">{{ p.total }}</h4>
              </b-badge>
            </b-button>
          </div>

          <b-collapse :id="'acc-' + p.id_pack" accordion="my-accordion" role="tabpanel">
            <div class="card-body">
              <ul class="list-group">
                <li class="list-group-item" v-for="k in p.kembali_pack">
                   <b-form-checkbox v-model="k.status" value="1" unchecked-value="0">
                    {{ k.pembeli.nama + " : " + k.jumlah }}
                  </b-form-checkbox>
                </li>
              </ul>
            </div>
          </b-collpase>
        </div>
      </div>
      <b-button class="mt-2" block variant="success" @click="verifyPack()"
      v-if="setorPack.length > 0">
        Verifikasi Pack
      </b-button>
  </div>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        driver: [],
        pack: [],
        setorPack: [],
        id_sopir: null,
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
      get_driver : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.post(base_url + "/driver")
        .then(response => {
          this.driver = response.data.driver;
          this.get_pack();
        })
        .catch(error => {
          alert(error);
        })
      },

      get_pack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/pack")
        .then(response => {
          response.data.pack.forEach((item, i) => {
            item.kembali_pack = [];
            item.total = 0;
          });

          this.pack = response.data.pack;
        })
        .catch(error => {
          alert(error);
        })
      },

      getSetoranPack : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/setoran-pack/" + this.id_sopir, conf)
        .then(response => {
          this.setorPack = response.data;
          this.grouping();
        })
        .catch(error => {
          console.log(error);
        })
      },

      grouping : function(){
        this.pack.forEach((item, i) => {
          let total = 0;
          item.kembali_pack = this.setorPack.filter(it => {
            if (it.id_pack === item.id_pack) total += Number(it.jumlah);
            it.status = 1;
            return it.id_pack === item.id_pack;
          });
          item.total = total;
        });
      },

      verifyPack : function(){
        if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {
          this.setorPack = [];
          this.pack.forEach((item, i) => {
            Array.prototype.push.apply(this.setorPack,item.kembali_pack);
          });

          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("kembali_pack", JSON.stringify(this.setorPack));
          axios.post(base_url + "/verify-pack", form, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.getSetoranPack();
          })
          .catch(error => {
            console.log(error);
          })
        }
      },
    },

    mounted(){
      this.get_driver();

    }
  }
</script>
