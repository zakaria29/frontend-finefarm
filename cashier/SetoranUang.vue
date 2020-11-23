<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Setoran Pack
      </h3>
    </div>
    <div class="card-body">
      Pilih Driver (Sopir)
      <v-select :options="driver" @input="getSetoranUang()"
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

      <b-alert class="mt-2" variant="warning" :show="id_sopir && setorUang.length === 0">
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

      <div class="mt-2" v-if="id_sopir && setorUang.length > 0">
        <h4>Verifikasi Setoran Uang: {{ "Rp " + formatNumber(getTotal(setorUang)) }}</h4>
        <ul class="mb-1 list-group">
          <li class="list-group-item" v-for="item in setorUang">
            <b-row>
              <b-col cols="5">
                <b-form-checkbox v-model="item.status" value="1" unchecked-value="0">
                  {{ item.pembeli.nama }}
               </b-form-checkbox>
                <small> <i>{{ "ID Order: " + item.id_orders }}</i> </small>
              </b-col>
              <b-col>{{ "Rp " + formatNumber(item.nominal) }}</b-col>
            </b-row>
          </li>
        </ul>
      </div>

      <b-button class="mt-2" block variant="success" @click="verifyUang()"
      v-if="setorUang.length > 0">
        Verifikasi Setoran Uang
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
        setorUang: [],
        id_sopir: null,
        message : "",
        key: "",
      }
    },

    methods: {
      get_driver : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.post(base_url + "/driver")
        .then(response => {
          this.driver = response.data.driver;
        })
        .catch(error => {
          alert(error);
        })
      },

      getTotal : function(arr){
        let total = 0;
        arr.forEach((item, i) => {
          if(item.status === "1")
          total += Number(item.nominal)
        });
        return total;
      },

      getSetoranUang : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/setoran-uang/" + this.id_sopir, conf)
        .then(response => {
          response.data.forEach((item, i) => {
            item.status = "1";
          });

          this.setorUang = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },

      verifyUang : function(){
        if (confirm("Apakah Anda yakin data ini telah terverifikasi?")) {

          this.$bvToast.show("loading");
          let conf = { headers: { "Api-Token" : this.key} };
          let form = new FormData();
          form.append("setor_uang", JSON.stringify(this.setorUang));
          axios.post(base_url + "/verify-uang", form, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.$bvToast.show("message");
            this.getSetoranUang();
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
