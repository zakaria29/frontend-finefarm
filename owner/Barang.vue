<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-cubes"></span> Data Barang
      </h3>
    </div>
    <div class="card-body" style="overflow:auto">
      <b-table striped hover :items="barangs" :fields="fields">
        <template v-slot:cell(satuan)="data">
          {{ data.item.satuan === "1" ? "Kilogram": "Butir" }}
        </template>
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item)"
          v-b-modal.modal_barang>
            <span class="fa fa-edit"></span>
          </b-button>
          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_barang)">
            <span class="fa fa-trash"></span>
          </b-button>
          <b-button class="btn btn-sm btn-success" @click="SetHarga(data.item)"
          v-b-modal.modal_set_harga>
            <span class="fa fa-cog"></span> Set Harga
          </b-button>

          <b-button class="btn btn-sm btn-warning" @click="LogHarga(data.item)"
          v-b-modal.modal_log>
            <span class="fa fa-clock"></span> Log Harga
          </b-button>
        </template>
        <template v-slot:cell(harga)="data">
          {{ "Rp " + formatNumber(data.item.harga) }}
        </template>
        <template v-slot:cell(stok)="data">
          {{data.item.stok + " " + (data.item.satuan === "1" ? "Kg" : "Butir") }}
        </template>
      </b-table>
    </div>
    <div class="card-footer">
      <b-button class="btn btn-sm btn-success" v-b-modal.modal_barang
      @click="Add">
      <span class="fa fa-plus"></span> Tambah Data
      </b-button>
    </div>

    <b-modal id="modal_set_harga" :title="title_modal"
      header-bg-variant="info"
      border-variant="info" hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="SaveHarga">
          Set harga
           <b-form-input v-model="barang.harga" require type="number" class="mb-1">
           </b-form-input>
           <b-button type="submit" class="btn btn-sm btn-success btn-block">
             <span class="fa fa-check"></span> Simpan
           </b-button>
        </form>
      </b-container>
    </b-modal>

    <b-modal id="modal_log" :title="title_modal"
      header-bg-variant="info"
      border-variant="info" hide-footer>

      <b-container fluid>
          <b-table striped hover
                 :items="logHarga"
                 :fields="['waktu','harga','diubah_oleh']">
            <template v-slot:cell(diubah_oleh)="data">
              {{ data.item.nama }}
            </template>
            <template v-slot:cell(harga)="data">
              {{ "Rp " + formatNumber(data.item.harga) }}
            </template>
        </b-table>
        </form>
      </b-container>
    </b-modal>

    <b-modal
     id="modal_barang"
     title="Form Barang"
     header-bg-variant="info"
     border-variant="info" hide-footer>

     <b-container fluid>
       <form v-on:submit.prevent="Save">
         Nama Barang
         <b-form-input v-model="barang.nama_barang" class="mb-1" required>
         </b-form-input>

         Satuan
         <b-form-select v-model="barang.satuan" class="mb-1">
           <option value="1">Kilogram</option>
           <option value="2">Butir</option>
         </b-form-select>

         Keterangan
         <b-form-select v-model="barang.keterangan" class="mb-1">
           <option value="1">Utuh</option>
           <option value="0">Tidak Utuh</option>
         </b-form-select>

         Pack
         <b-table class="mb-1" :items="pack_barang"
         :fields="['nama_pack','kapasitas_kg','kapasitas_butir','option']">
           <template v-slot:cell(option)="data">
             <b-button class="btn btn-sm btn-danger" @click="Hapus(data.index)">
               <span class="fa fa-trash"></span>
             </b-button>
           </template>

           <template v-slot:cell(kapasitas_kg)="data">
              <b-form-input v-model="data.item.kapasitas_kg" type="number" step="0.01">
              </b-form-input>
              per 10Kg
           </template>
           <template v-slot:cell(kapasitas_butir)="data">
              <b-form-input v-model="data.item.kapasitas_butir" type="number" step="0.01">
              </b-form-input>
              per pack
           </template>
         </b-table>
         <small>Tambah Pack ke Barang</small>
         <b-row>
           <div class="col-9">
             <b-form-select v-model="selectedPack" class="mb-1">
               <option v-for="item in pack" :value="item.id_pack">
                 {{ item.nama_pack }}
               </option>
             </b-form-select>
           </div>
           <div class="col-3">
             <b-button class="btn btn-sm btn-success" @click="TambahPack">
               Tambahkan
             </b-button>
           </div>
         </b-row>

         <b-button type="submit" variant="success" size="md" class="pull-right btn-block">
            <span class="fa fa-check"></span> Simpan
          </b-button>
       </form>
     </b-container>
   </b-modal>

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
      action : "",
      barang : {
        id_barang: "",
        nama_barang: "",
        keterangan: "",
        satuan: "",
        harga: 0,
      },
      pack_barang: [],
      key : "",
      message: "",
      barangs: [],
      pack: [],
      logHarga: [],
      selectedPack : "",
      title_modal: "",
      fields: ["nama_barang","satuan","stok","harga","option"],
    }
  },

  methods : {
    SetHarga : function(item){
      this.title_modal = "Set Harga " + item.nama_barang;
      this.barang.id_barang = item.id_barang;
      this.barang.harga = 0;
    },
    LogHarga : function(item){
      this.title_modal = "Log Harga " + item.nama_barang;
      this.logHarga = item.log_harga_barang;
    },
    TambahPack : function(){
      if (! this.pack_barang.some(el => el.id_pack === this.selectedPack)) {
        let item = this.pack.find(elm => elm.id_pack === this.selectedPack);
        this.pack_barang.push({
          id_pack : item.id_pack,
          nama_pack: item.nama_pack,
          keterangan: item.keterangan,
          kapasitas_kg: 0,
          kapasitas_butir: 0,
        });
      }
    },

    Hapus : function(index){
      this.pack_barang.splice(index,1);
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

    get_barang : function(){
      let conf = { headers: { "Api-Token" : this.key} };
      axios.get(base_url + "/barang", conf)
      .then(response => {
        this.barangs = response.data.barang;
        this.$bvToast.hide("loading");
      })
      .catch(error => {
        console.log(error);
      })
    },

    Add : function(){
      this.action = "insert";
      this.barang.id_barang = "";
      this.barang.nama_barang = "";
      this.barang.keterangan = 1;
      this.barang.satuan = "";
      this.pack_barang = [];
    },

    Edit : function(item){
      this.action = "update";
      this.barang.id_barang = item.id_barang;
      this.barang.nama_barang = item.nama_barang;
      this.barang.keterangan = item.keterangan;
      this.barang.satuan = item.satuan;
      this.pack_barang = item.pack;
    },

    Save : function(){
      this.$bvToast.show("loading");
      this.$bvModal.hide("modal_barang");
      let conf = { headers: { "Api-Token" : this.key} };
      let form = new FormData();
      form.append("id_barang", this.barang.id_barang);
      form.append("nama_barang", this.barang.nama_barang);
      form.append("keterangan", this.barang.keterangan);
      form.append("satuan", this.barang.satuan);
      form.append("pack_barang", JSON.stringify(this.pack_barang));


      let useUrl = "";
      if (this.action === "insert") {
        useUrl = base_url + "/barang/save";
      }else{
        useUrl = base_url + "/barang/update";
      }

      axios.post(useUrl, form, conf)
      .then(response => {
        this.$bvToast.hide("loading");
        this.message = response.data.message;
        this.get_barang();
        this.$bvToast.show("message");
      })
      .catch(error => {
        console.log(error);
      })
    },

    SaveHarga : function(){
      this.$bvToast.show("loading");
      this.$bvModal.hide("modal_set_harga");
      let conf = { headers: { "Api-Token" : this.key} };
      let form = new FormData();
      form.append("id_barang", this.barang.id_barang);
      form.append("harga", this.barang.harga);
      form.append("id_users","IDU69098"); // sementara


      let useUrl = base_url + "/barang/update_harga";

      axios.post(useUrl, form, conf)
      .then(response => {
        this.$bvToast.hide("loading");
        this.message = response.data.message;
        this.get_barang();
        this.$bvToast.show("message");
      })
      .catch(error => {
        console.log(error);
      })
    },

    Drop : function(id){
      let conf = { headers: { "Api-Token" : this.key} };
      if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
        $('#loading').toast('show');
        axios.delete(base_url + "/barang/drop/"+id, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.message = response.data.message;
          this.get_barang();
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        })
      }
    },
  },

  mounted(){
    this.$bvToast.show("loading");
    this.get_barang();
    this.get_pack();

  }
}
</script>
