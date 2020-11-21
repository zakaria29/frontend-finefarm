<template>
  <div class="card">
    <div class="card-header">
      <h3>
        <span class="fa fa-list"></span> List Supply
      </h3>
    </div>

    <div class="card-body">
      <form v-on:submit.prevent="find">
        <div class="row mb-2">
          <div class="col-2">
            Start Date
             <b-form-input v-model="from" type="date" required>
             </b-form-input>
          </div>
          <div class="col-2">
            End Date
             <b-form-input v-model="to" type="date" required>
             </b-form-input>
          </div>
          <div class="col-6">
            Pencarian
             <b-form-input v-model="search" type="text" placeholder="Kata Kunci">
             </b-form-input>
          </div>
          <div class="col-2">
            <br />
            <b-button type="submit" class="btn btn-info">
              <span class="fa fa-search"></span> Cari
            </b-button>
          </div>
        </div>
      </form>

      <b-table striped hover :items="supply" :fields="fields">
        <template v-slot:cell(penerima)="data">
          {{ data.item.nama }}
        </template>
        <template v-slot:cell(total)="data">
          {{ "Rp " + formatNumber(data.item.total_bayar) }}
        </template>
        <template v-slot:cell(option)="data">
          <b-button class="btn btn-sm btn-info" @click="Edit(data.item.id_supply)">
            <span class="fa fa-edit"></span>
          </b-button>

          <b-button class="btn btn-sm btn-danger" @click="Drop(data.item.id_supply)">
            <span class="fa fa-trash"></span>
          </b-button>

          <b-button class="btn btn-sm btn-warning" @click="Detail(data.item)"
          v-b-modal.modal_detail>
            <span class="fa fa-list"></span> Detail Supply
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

    <b-modal id="modal_detail" title="Detai Supply"
      header-bg-variant="info" size="lg"
      border-variant="info" hide-footer>

      <b-container fluid>
          <b-table striped hover :items="detail_supply"
          :fields="['nama_barang','harga_beli','utuh','bentes','putih','pecah','loss']">
            <template v-slot:cell(utuh)="data">
              {{ data.item.jumlah_utuh }}
            </template>
            <template v-slot:cell(bentes)="data">
              {{ data.item.jumlah_bentes }}
            </template>
            <template v-slot:cell(pecah)="data">
              {{ data.item.jumlah_pecah }}
            </template>
            <template v-slot:cell(putih)="data">
              {{ data.item.jumlah_putih }}
            </template>
            <template v-slot:cell(loss)="data">
              {{ data.item.jumlah_loss }}
            </template>
            <template v-slot:cell(harga_beli)="data">
              {{ "Rp " + formatNumber(data.item.harga_beli) }}
            </template>
        </b-table>
        </form>
      </b-container>
    </b-modal>

  </div>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        message: "",
        key: "",
        supply: [],
        totalRow: 0,
        currentPage: 1,
        perPage: 10,
        search: "",
        from: "",
        to: "",
        fields: ["waktu", "nama_supplier","penerima","total","option"],
        detail_supply: [],
      }
    },

    methods : {
      Edit : function(id){
        this.$router.push({ name: "EditSupply", params: { id_supply: id }});

      },
      Detail : function(item){
        this.detail_supply = item.detail_supply;
      },

      Drop : function(id){
        let conf = { headers: { "Api-Token" : this.key} };
        if (confirm("Apakah Anda yakin ingin menghapus data ini?")) {
          this.$bvToast.show("loading");
          axios.delete(base_url + "/supply/drop/" + id, conf)
          .then(response => {
            this.$bvToast.hide("loading");
            this.message = response.data.message;
            this.find();
            this.$bvToast.show("message");
          })
          .catch(error => {
            alert(error);
          })
        }
      },
      find : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.currentPage-1) * this.perPage;
        this.$bvToast.show("loading");
        let form = new FormData();
        form.append("find", this.search);
        form.append("from", this.from);
        form.append("to", this.to);

        axios.post(base_url + "/supply/"+this.perPage+"/"+offset, form, conf)
        .then(response => {
          this.$bvToast.hide("loading");
          this.supply = response.data.supply;
          this.totalRow = response.data.count;
        })
        .catch(error => {
          alert(error);
        })
      }
    },

    mounted(){
      let date = new Date();
      let currentDate = date.toISOString().slice(0,10);
      this.to = currentDate;
      let lastMonth = new Date(date.getFullYear(), date.getMonth()-1, date.getDate());
      this.from = lastMonth.toISOString().slice(0,10);

      this.find();
    }
  }
</script>
