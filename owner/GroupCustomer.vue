<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-id-card-alt"></span>
        Data Group Customer
      </h3>
    </div>
    <div class="card-body">
      <b-table striped hover :items="groups" :fields="fields">
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item)"
          v-b-modal.modal_group>
            <span class="fa fa-edit"></span>
          </b-button>
          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_group_customer)">
            <span class="fa fa-trash"></span>
          </b-button>
        </template>
      </b-table>

    </div>
    <div class="card-footer">
      <b-button class="btn btn-sm btn-success" v-b-modal.modal_group
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
     id="modal_group"
     title="Form Group Customer"
     header-bg-variant="info"
     border-variant="info" hide-footer>

     <b-container fluid>
       <form v-on:submit.prevent="Save">
         Nama Group
         <b-form-input v-model="group.nama_group_customer" class="mb-1" required>
         </b-form-input>

         Margin
         <b-form-input v-model="group.margin" type="number" class="mb-1" required>
         </b-form-input>

         <b-button type="submit" variant="success" size="md" class="pull-right">
            <span class="fa fa-check"></span> Simpan
          </b-button>
       </form>
     </b-container>
 </b-modal>
  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data: function(){
      return {
        fields: ["nama_group_customer","margin","option"],
        action: "",
        group: {
          id_group_customer: "",
          nama_group_customer: "",
          margin: 0,
        },
        message: "",
        groups: [],
      }
    },

    methods: {
      get_group_customer : function() {
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loading");
        axios.get(base_url+"/group_customer", conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.groups = response.data.group_customer;
        })
        .catch(error => {
          console.log(error);
        });
      },

      Add : function(){
        this.action = "insert";
        this.group.id_group_customer = "";
        this.group.nama_group_customer = "";
        this.group.margin = 0;
      },

      Edit : function(item){
        this.action = "update";
        this.group.id_group_customer = item.id_group_customer;
        this.group.nama_group_customer = item.nama_group_customer;
        this.group.margin = item.margin;
      },

      Save : function(){
        this.$bvModal.hide("modal_group");
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("id_group_customer", this.group.id_group_customer);
        form.append("nama_group_customer", this.group.nama_group_customer);
        form.append("margin", this.group.margin);
        form.append("action", this.action);
        let useUrl = base_url+"/group_customer";

        axios.post(useUrl, form, conf)
        .then(response => {
          this.message = response.data.message;
          this.get_group_customer();
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
          axios.delete(base_url + "/group_customer/"+id, conf)
          .then(response => {
            this.message = response.data.message;
            this.get_group_customer();
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
      this.get_group_customer();
    }
  }
</script>
