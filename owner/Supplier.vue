<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-people-carry"></span>
        Data Supplier
      </h3>
    </div>
    <div class="card-body" style="overflow:auto">
      <form v-on:submit.prevent="find">
         <b-form-input v-model="search" class="mb-1" placeholder="Pencarian...">
         </b-form-input>
      </form>
      <b-table striped hover :items="suppliers" :fields="fields">
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item)"
          v-b-modal.modal_supplier>
            <span class="fa fa-edit"></span>
          </b-button>
          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_supplier)">
            <span class="fa fa-trash"></span>
          </b-button>
          <b-button class="btn btn-sm btn-success" @click="Stok(data.item)"
          v-b-modal.modal_stok>
            <span class="fa fa-cubes"></span> Stok Barang
          </b-button>

        </template>
      </b-table>

      <b-pagination
      size="md"
      class="mt-3"
      :total-rows="totalRow"
      v-model="currentPage"
      :per-page="perPage"
      align="center"
      v-on:input="find">
      </b-pagination>
    </div>
    <div class="card-footer">
      <b-button class="btn btn-sm btn-success" v-b-modal.modal_supplier
      @click="Add">
      <span class="fa fa-plus"></span> Tambah Data
      </b-button>
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

    <!-- Modal Component -->
    <b-modal
     id="modal_supplier"
     title="Form Supplier"
     header-bg-variant="info"
     border-variant="info" hide-footer>

     <b-container fluid>
       <form v-on:submit.prevent="Save">
         Nama Supplier
         <b-form-input v-model="supplier.nama_supplier" class="mb-1" required>
         </b-form-input>
         Alamat
         <b-form-input v-model="supplier.alamat" class="mb-1" required>
         </b-form-input>
         Kontak
         <b-form-input v-model="supplier.kontak" class="mb-1" required>
         </b-form-input>

         <b-button type="submit" variant="success" size="md" class="pull-right">
            <span class="fa fa-check"></span> Simpan
          </b-button>
       </form>
     </b-container>
 </b-modal>

 <b-modal id="modal_stok" :title="title_stok" hide-footer>
   <b-container fluid>
     <b-table striped hover :items="stok_barang" :fields="['nama_barang','stok']">
       <template v-slot:cell(stok)="data">
         {{ data.item.stok + " " + (data.item.satuan === "1" ? "Kg" : "Butir") }}
       </template>
     </b-table>
   </b-container>
 </b-modal>
  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data: function(){
      return {
        fields: ["nama_supplier","alamat","kontak","option"],
        action: "",
        supplier: {
          id_supplier: "",
          nama_supplier: "",
          alamat: "",
          kontak: "",
        },
        search : "",
        message: "",
        suppliers: [],
        totalRow: 0,
        perPage: 10,
        currentPage: 1,
        key: "",
        title_stok: "",
        stok_barang: []
      }
    },

    methods: {
      Stok : function(item){
        this.title_stok = "Stok dari " + item.nama_supplier;
        this.stok_barang = item.stok_barang;
      },
      get_supplier : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        axios.get(base_url+"/supplier/"+this.perPage+"/"+offset, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.suppliers = response.data.supplier;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      find : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        let form = new FormData();
        form.append("find", this.search);
        axios.post(base_url+"/supplier/"+this.perPage+"/"+offset, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.suppliers = response.data.supplier;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      Add : function(){
        this.action = "insert";
        this.supplier.id_supplier = "";
        this.supplier.nama_supplier = "";
        this.supplier.alamat = "";
        this.supplier.kontak = "";
      },

      Edit : function(item){
        this.action = "update";
        this.supplier.id_supplier = item.id_supplier;
        this.supplier.nama_supplier = item.nama_supplier;
        this.supplier.alamat = item.alamat;
        this.supplier.kontak = item.kontak;
      },

      Save : function(){
        this.$bvModal.hide("modal_supplier");
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("id_supplier", this.supplier.id_supplier);
        form.append("nama_supplier", this.supplier.nama_supplier);
        form.append("alamat", this.supplier.alamat);
        form.append("kontak", this.supplier.kontak);


        let useUrl = "";
        if (this.action === "insert") {
          useUrl = base_url + "/supplier/save";
        }else{
          useUrl = base_url + "/supplier/update";
        }

        axios.post(useUrl, form, conf)
        .then(response => {
          this.message = response.data.message;
          this.find();
          this.$bvToast.hide("loading");
          this.$bvToast.show("message");
        })
        .catch(error => {
          alert(error);
        })
      },

      Drop : function(id){
        let conf = { headers: { "Api-Token" : this.key} };
        if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
          this.$bvToast.show("loading");
          axios.delete(base_url + "/supplier/drop/"+id, conf)
          .then(response => {
            this.message = response.data.message;
            this.find();
            this.$bvToast.hide("loading");
            this.$bvToast.show("message");
          })
          .catch(error => {
            alert(error);
          })
        }
      },
    },

    mounted(){
      this.get_supplier();
    }
  }
</script>
