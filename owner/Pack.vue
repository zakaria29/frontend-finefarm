<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-box"></span> Data Pack
      </h3>
    </div>
    <div class="card-body">
      <b-table striped hover :items="package" :fields="fields">
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item)"
          v-b-modal.modal_pack>
            <span class="fa fa-edit"></span>
          </b-button>
          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_pack)">
            <span class="fa fa-trash"></span>
          </b-button>
          <br />
          <small class="text-info">* hanya untuk pack tertentu</small>
          <br />
          <b-button size="sm" variant="warning" v-b-modal.modal_kapasitas
          @click="EditKapasitas(data.item, '1')">
            Kapasitas dalam Kg
          </b-button>
          <b-button size="sm" variant="dark" v-b-modal.modal_kapasitas
          @click="EditKapasitas(data.item, '2')">
            Kapasitas satuan Butir
          </b-button>
        </template>
        <template v-slot:cell(harga)="data">
          {{ "Rp " + formatNumber(data.item.harga) }}
        </template>
      </b-table>
    </div>
    <div class="card-footer">
      <b-button class="btn btn-sm btn-success" v-b-modal.modal_pack
      @click="Add">
      <span class="fa fa-plus"></span> Tambah Data
      </b-button>
    </div>

    <b-modal
     id="modal_pack"
     title="Form Pack"
     header-bg-variant="info"
     border-variant="info" hide-footer>

     <b-container fluid>
       <form v-on:submit.prevent="Save">
         Nama Pack
         <b-form-input v-model="pack.nama_pack" class="mb-1" required>
         </b-form-input>
         Stok
         <b-form-input type="number" v-model="pack.stok" class="mb-1" required>
         </b-form-input>
         Harga
         <b-form-input type="number" v-model="pack.harga" class="mb-1" required>
         </b-form-input>
         Keterangan
         <b-form-select v-model="pack.keterangan" class="mb-1">
           <option value="0">Otomatis Diberikan kepada Customer</option>
           <option value="1">Otomatis Dipinjamkan kepada Customer</option>
         </b-form-select>

         <b-button type="submit" variant="success" size="md" class="pull-right">
            <span class="fa fa-check"></span> Simpan
          </b-button>
       </form>
     </b-container>
   </b-modal>

   <b-modal
    id="modal_kapasitas"
    title="Kapasitas Pack"
    header-bg-variant="info" size="lg"
    border-variant="info" hide-footer>

    <b-container fluid v-if="selectedPack !== null">
      <form v-on:submit.prevent="SaveKapasitasPack">
        Nama Pack: {{ selectedPack.nama_pack }}
        <ul class="list-group">
          <li class="list-group-item">
            <b-row>
              <b-col cols="5"> <strong>
                Kapasitas {{ (selectedPack.satuan === '1' ? ' (Kg)' : ' (Butir)') }}
              </strong> </b-col>
              <b-col cols="5"> <strong>Jumlah</strong> </b-col>
              <b-col>
                <b-button variant="success" @click="AddKapasitas" size="sm">
                  <span class="fa fa-plus"></span>
                </b-button>
              </b-col>
            </b-row>
          </li>
          <li class="list-group-item" v-for="(item,i) in selectedPack.kapasitas">
            <b-row>
              <b-col cols="5">
                <b-form-input v-model="item.kapasitas" size="sm" type="number" required min="1" />
              </b-col>
              <b-col cols="5">
                <b-form-input v-model="item.jumlah" size="sm" type="number" required min="1" />
              </b-col>
              <b-col>
                <b-button variant="danger" @click="RemoveKapasitas(i)" size="sm">
                  <span class="fa fa-trash"></span>
                </b-button>
              </b-col>
            </b-row>
          </li>
        </ul>
        <b-button type="submit" variant="success" size="md" block>
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
      pack : {
        id_pack: "",
        nama_pack: "",
        stok: 0,
        keterangan: "",
        harga: 0
      },
      key : "",
      message: "",
      package: [],
      fields: ["nama_pack","stok","harga","option"],
      selectedPack: null,
    }
  },

  methods : {
    get_pack : function(){
      let conf = { headers: { "Api-Token" : this.key} };
      axios.get(base_url + "/pack", conf)
      .then(response => {
        response.data.pack.forEach((item, i) => {
          item.kapasitas = [];
          item.satuan = "";
        });

        this.package = response.data.pack;
        this.$bvToast.hide("loading");
      })
      .catch(error => {
        alert(error);
      })
    },

    EditKapasitas : function(item, satuan){
      this.selectedPack = item;
      if (satuan === "1") {
        this.selectedPack.satuan = "1";
        this.selectedPack.kapasitas = this.selectedPack.kapasitas_kg;
      }else if(satuan === "2"){
        this.selectedPack.satuan = "2";
        this.selectedPack.kapasitas = this.selectedPack.kapasitas_butir;
      }
    },

    AddKapasitas : function(){
      this.selectedPack.kapasitas.push({
        jumlah: 0,
        kapasitas: 0,
        satuan: this.selectedPack.satuan
      })
    },

    RemoveKapasitas : function(index){
      this.selectedPack.kapasitas.splice(index,1);
    },

    SaveKapasitasPack : function(){
      this.$bvToast.show("loading");
      this.$bvModal.hide("modal_kapasitas");
      let conf = { headers: { "Api-Token" : this.key} };
      let form = new FormData();
      form.append("id_pack", this.selectedPack.id_pack);
      form.append("satuan", this.selectedPack.satuan);
      form.append("kapasitas_pack", JSON.stringify(this.selectedPack.kapasitas));

      let useUrl = base_url + "/pack/kapasitas";
      axios.post(useUrl, form, conf)
      .then(response => {
        this.$bvToast.hide("loading");
        this.message = response.data.message;
        this.get_pack();
        this.$bvToast.show("message");
      })
      .catch(error => {
        console.log(error);
      })
    },

    Add : function(){
      this.action = "insert";
      this.pack.id_pack = "";
      this.pack.nama_pack = "";
      this.pack.stok = 0;
      this.pack.keterangan = "0";
      this.pack.harga = 0;
    },

    Edit : function(item){
      this.action = "update";
      this.pack.id_pack = item.id_pack;
      this.pack.nama_pack = item.nama_pack;
      this.pack.stok = item.stok;
      this.pack.keterangan = item.keterangan;
      this.pack.harga = item.harga;
    },

    Save : function(){
      this.$bvToast.show("loading");
      this.$bvModal.hide("modal_pack");
      let conf = { headers: { "Api-Token" : this.key} };
      let form = new FormData();
      form.append("id_pack", this.pack.id_pack);
      form.append("nama_pack", this.pack.nama_pack);
      form.append("keterangan", this.pack.keterangan);
      form.append("stok", this.pack.stok);
      form.append("harga", this.pack.harga);


      let useUrl = "";
      if (this.action === "insert") {
        useUrl = base_url + "/pack/save";
      }else{
        useUrl = base_url + "/pack/update";
      }

      axios.post(useUrl, form, conf)
      .then(response => {
        this.$bvToast.hide("loading");
        this.message = response.data.message;
        this.get_pack();
        this.$bvToast.show("message");
      })
      .catch(error => {
        alert(error);
      })
    },

    Drop : function(id){
      let conf = { headers: { "Api-Token" : this.key} };
      if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
        $('#loading').toast('show');
        axios.delete(base_url + "/pack/drop/"+id, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.message = response.data.message;
          this.get_pack();
          this.$bvToast.show("message");
        })
        .catch(error => {
          alert(error);
        })
      }
    },
  },

  mounted(){
    this.$bvToast.show("loading");
    this.get_pack();

  }
}
</script>
