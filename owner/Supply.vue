<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-download"></span> Supply Barang
      </h3>
    </div>

    <div class="card-body">
      <form v-on:submit.prevent="Save">
        <div class="row mb-2">
          <div class="col-sm-4">
            Pilih Supplier
          </div>
          <div class="col-sm-8">
            <b-form-select v-model="supply.id_supplier" required>
              <option v-for="s in supplier" :value="s.id_supplier">
                {{ s.nama_supplier }}
              </option>
            </b-form-select>
          </div>
        </div>

        <div class="row mb-2">
          <div class="col-sm-4">
            Waktu
          </div>
          <div class="col-sm-4">
            <input type="date" v-model="supply.date" class="form-control" required />
          </div>
          <div class="col-sm-2">
            <input type="time" v-model="supply.time" class="form-control" required />
          </div>
        </div>

        <div class="row mb-2">
          <h4>List Barang</h4>
          <ul class="list-group">
            <li v-for="(detail, index) in detail_supply" class="list-group-item mb-2">
              <div class="row">
                <div class="col-4">
                  Pilih Barang
                  <b-form-select v-model="detail.id_barang" required>
                    <option v-for="b in barang" :value="b.id_barang">
                      {{ b.nama_barang }}
                    </option>
                  </b-form-select>
                </div>
                <div class="col-2">
                  Jumlah Utuh
                   <b-form-input v-model="detail.jumlah_utuh" type="number" required>
                   </b-form-input>
                </div>
                <div class="col-6">
                  Harga Beli
                   <b-form-input v-model="detail.harga_beli" type="number" required>
                   </b-form-input>
                </div>

                <div class="col-3">
                  Jumlah Bentes
                   <b-form-input v-model="detail.jumlah_bentes" type="number" required>
                   </b-form-input>
                </div>
                <div class="col-3">
                  Jumlah Pecah
                   <b-form-input v-model="detail.jumlah_pecah" type="number" required>
                   </b-form-input>
                </div>
                <div class="col-3">
                  Jumlah Putih
                   <b-form-input v-model="detail.jumlah_putih" type="number" required>
                   </b-form-input>
                </div>
                <div class="col-3">
                  Jumlah Loss
                   <b-form-input v-model="detail.jumlah_loss" type="number" required>
                   </b-form-input>
                </div>

                <b-button @click="Drop(index)"
                class="btn btn-danger btn-sm btn-block mt-2">
                  <span class="fa fa-trash"></span> Hapus
                </b-button>
              </div>
            </li>
          </ul>
        </div>

        <b-button class="btn btn-info mb-2" @click="Add">
          <span class="fa fa-plus"></span> Tambah List
        </b-button>

        <div class="row mb-2">
          <div class="col-4">
            Total Bayar
          </div>
          <div class="col-8">
             <b-form-input v-model="supply.total_bayar" type="number" min="1" required>
             </b-form-input>
          </div>
        </div>

        <div class="row mb-2">
          <b-button class="btn btn-success btn-block" type="submit">
            <span class="fa fa-check"></span> Simpan
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
        message : "",
        key : "",
        barang : [],
        supplier: [],
        supply : {
          id_supply: "",
          id_supplier: "",
          date: "",
          time: "",
          id_users : "",
          total_bayar: 0
        },
        detail_supply: [],
      }
    },

    methods : {
      Add : function(){
        this.detail_supply.push({
          id_barang: "",
          harga_beli: 0,
          jumlah_utuh: 0,
          jumlah_bentes: 0,
          jumlah_putih: 0,
          jumlah_loss: 0,
          jumlah_pecah: 0
        });
      },
      Drop : function(index){
        this.detail_supply.splice(index,1);
      },
      get_barang : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/barang", conf)
        .then(response => {
          response.data.barang.map(it => {
            if (it.keterangan === "1") {
              this.barang.push(it);
            }
          });

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

      Save : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        if (this.detail_supply.length > 0) {
          if (confirm("Apakah Anda yakin menyimpan data supply ini?")) {
            this.$bvToast.show("loading");
            let form = new FormData();
            form.append("id_supply", this.supply.id_supply);
            form.append("waktu", this.supply.date + " " + this.supply.time);
            form.append("id_supplier", this.supply.id_supplier);
            form.append("total_bayar", this.supply.total_bayar);
            form.append("id_users", "IDU69098"); // sementara
            form.append("detail_supply", JSON.stringify(this.detail_supply));

            axios.post(base_url + "/supply/save", form, conf)
            .then(response => {
              this.$bvToast.hide("loading");
              this.message = response.data.message;
              this.$bvToast.show("message");
              this.refersh();
            })
            .catch(error => {
              alert(error);
            })
          }
        }else{
          alert("Anda belum memilih barang yang disupply");
        }
      },

      refersh : function(){
        this.detail_supply = [];
        this.supply.id_supply = "";
        this.supply.id_supplier = "";
        this.supply.total_bayar = 0;

        let date = new Date();
        let currentDate = date.toISOString().slice(0,10);
        let currentTime = date.getHours() + ':' + date.getMinutes();
        this.supply.date = currentDate;
        // this.supply.time = currentTime;
        this.supply.time = (date.getHours() < 10 ? "0"+date.getHours() : date.getHours())
        + ":" + (date.getMinutes() < 10 ? "0"+date.getMinutes() : date.getMinutes());
      },
    },

    mounted(){
      this.get_supplier();
      this.get_barang();
      let date = new Date();
      let currentDate = date.toISOString().slice(0,10);
      let currentTime = date.getHours() + ':' + date.getMinutes();
      this.supply.date = currentDate;
      // this.supply.time = currentTime;
      this.supply.time = (date.getHours() < 10 ? "0"+date.getHours() : date.getHours())
      + ":" + (date.getMinutes() < 10 ? "0"+date.getMinutes() : date.getMinutes());
    }
  }
</script>
