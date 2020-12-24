<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> Data Tanggungan Pack
      </h3>
    </div>
    <div class="card-body">

      <b-toast id="message" title="Message" solid>
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <b-toast id="loading" title="Processing Data" solid no-auto-hide>
        <!-- <b-spinner label="Spinning" variant="success"></b-spinner> -->
        <span class="fa fa-spinner fa-spin"></span>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <b-row class="mb-1">
        <b-col>
          <form v-on:submit.prevent="getTanggungan">
             <b-form-input v-model="search" class="mb-1" placeholder="Pencarian...">
             </b-form-input>
          </form>
        </b-col>
      </b-row>
      <b-alert class="mt-2" variant="warning" :show="tanggungan.length === 0">
        Tidak Ada Tanggungan Pack
      </b-alert>
      <div class="mt-2" v-if="tanggungan.length > 0">
        <ul class="list-group">
          <li class="list-group-item" v-for="item in tanggungan">
            <h3>{{ item.nama }}</h3>
            <h4>Tanggungan Pack:</h4>
            <b-row>
              <b-col cols="4" v-for="it in item.tanggungan_pack"
              v-if="item.tanggungan_pack.length > 0">
                <strong class="text-info">{{ it.pack.nama_pack }}</strong>
                <strong>: {{ it.jumlah }} <small>item</small> </strong>
              </b-col>
            </b-row>
            <b-button pill size="sm" variant="primary" @click="editTanggunganPack(item)"
            v-b-modal.modal_tanggungan>
               <span class="fa fa-edit"></span> Edit Tanggungan Pack
            </b-button>
          </li>
        </ul>
      </div>
  </div>

  <b-modal id="modal_tanggungan" title="Edit Tanggungan Pack"
    header-bg-variant="success" size="lg"
    border-variant="info" hide-footer>
    <b-container fluid v-if="selectedUser !== null">
      <form v-on:submit.prevent="saveTanggungan">
        <ul class="list-group">
          <li class="list-group-item">
            <b-row>
              <b-col cols="6">Pack</b-col>
              <b-col cols="4">Jumlah</b-col>
              <b-col cols="2">
                <b-button variant="success" size="sm" @click="addTanggungan">
                  <span class="fa fa-plus"></span>
                </b-button>
              </b-col>
            </b-row>
          </li>
          <li class="list-group-item" v-for="(item, index) in tanggungan_pack">
            <b-row>
              <b-col cols="6">
                <b-form-select v-model="item.id_pack" required>
                  <option v-for="it in pack" :value="it.id_pack">
                    {{ (it.id_pack === null) ? "" : it.nama_pack }}
                  </option>
                </b-form-select>
              </b-col>
              <b-col cols="4">
                <b-form-input v-model="item.jumlah" required type="number" />
              </b-col>
              <b-col cols="2">
                <b-button variant="danger" size="sm" @click="removeTanggungan(index)">
                  <span class="fa fa-trash"></span>
                </b-button>
              </b-col>
            </b-row>
          </li>
        </ul>
        <b-button block type="submit" variant="primary" size="sm">
          Simpan
        </b-button>
      </form>
    </b-container>
  </b-modal>
</div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        users: null,
        pack: [],
        tanggungan: [],
        message : "",
        search: "",
        key: "",
        selectedUser: null,
        tanggungan_pack: [],
      }
    },

    methods: {
      editTanggunganPack : function(item){
        this.selectedUser = item;
        this.tanggungan_pack = [];
        item.tanggungan_pack.forEach((it, i) => {
          this.tanggungan_pack.push({
            id_pack: it.id_pack,
            jumlah: it.jumlah
          })
        });

      },

      addTanggungan : function(){
        this.tanggungan_pack.push({
          id_pack: null,
          jumlah: 0
        });
      },

      removeTanggungan : function(index){
        this.tanggungan_pack.splice(index, 1);
      },

      saveTanggungan : function(){
        this.$bvToast.show("loading");
        this.$bvModal.hide("modal_tanggungan");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("id_users", this.selectedUser.id_users);
        form.append("tanggungan_pack", JSON.stringify(this.tanggungan_pack));

        axios.post(base_url + "/save-customer-pack", form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.message = response.data.message;
          this.$bvToast.show("message");
          this.getTanggungan();
        })
        .catch(error => {
          console.log(error);
        })
      },

      getTanggungan : function(){
        this.$bvToast.show("loading");
        let conf = { headers: { "Api-Token" : this.key} };
        let form = new FormData();
        form.append("find", this.search);

        axios.post(base_url + "/customer-pack", form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          let data = response.data.filter(it => it.tanggungan_pack.length > 0);
          this.tanggungan = data;
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
          this.getTanggungan();
        })
        .catch(error => {
          console.log(error);
        })
      },

      init : async function(){
        await this.get_pack();
      },
    },

    mounted(){
      this.init();
    }
  }
</script>
