<template>
  <div style="max-width: 100%">
    <b-overlay :show="uploading">
      <b-modal hide-footer hide-header centered id="item-modal">
        <div style="padding: 20px; background-color: white">
          <div class="container-fluid">
            <p>
              <strong>Tambah Barang {{ invItem ? "" : "Retur" }}</strong>
            </p>

            <form role="form">
              <div class="form-inline justify-content-between">
                <label class="mb-2">Nama</label>
                <input
                  type="text"
                  class="form-control mb-2"
                  list="barang-list"
                  :placeholder="'Nama Barang'"
                  v-model="tmpItem['namaBarang']"
                />
                <datalist id="barang-list">
                  <option
                    v-for="(item, index) in itemHelper"
                    :key="item + index"
                  >
                    {{ item }}
                  </option>
                </datalist>
              </div>

              <div class="form-inline justify-content-between">
                <label class="mb-2">Jumlah</label>
                <input
                  type="number"
                  class="form-control mb-2"
                  :placeholder="'Masukkan Jumlah'"
                  v-model="tmpItem['stock']"
                />
              </div>

              <div class="form-inline justify-content-between" role="form">
                <label class="mb-2">Harga</label>
                <input
                  type="number"
                  class="form-control mb-2"
                  :placeholder="'Masukkan Harga'"
                  v-model="tmpItem['harga']"
                />
              </div>
            </form>

            <button
              type="button"
              class="btn btn-danger modalBtn"
              @click="
                () => {
                  $bvModal.hide('item-modal');
                }
              "
              style="margin-left: 10px; margin-top: 10px"
            >
              Batal
            </button>
            <button
              type="button"
              class="btn btn-success modalBtn"
              style="margin-top: 10px"
              @click="addInvoiceItem"
            >
              Tambah
            </button>
          </div>
        </div>
      </b-modal>

      <b-modal hide-footer hide-header centered id="error-modal">
        <div
          class="justify-content-center align-items-center d-flex"
          style="background-color: white; padding: 20px"
        >
          {{ errorMessage }}
        </div>
      </b-modal>

      <div
        :style="{
          backgroundImage: 'url(' + tabImage + ')',
          backgroundSize: '80% 250%',
          backgroundRepeat: 'no-repeat',
          padding: '15px',
        }"
      >
        <h1 style="color: white">{{ $appName }}</h1>
      </div>

      <div id="printPage">
        <div id="content-area">
          <!-- <div id="merk">
            <h3 style="color: gray">Blessing</h3>
          </div> -->
          <b-container fluid class="px-0">
            <b-row class="basicInformation m-0" no-gutters>
              <b-col class="pt-1">
                <label for="namaToko">Nama Toko :</label>
              </b-col>
              <b-col>
                <b-form-input v-if="isViewMode"
                  id="namaToko"
                  :style="{ textAlign: isViewMode ? 'right' : 'left' }"
                  size="md"
                  type="text"
                  v-model="namaPembeli"
                  :plaintext="isViewMode"
                  :readonly="isViewMode"
                />
                <select v-else :options="optionsPembeli" v-model="namaPembeli" class="form-control" id="namaToko">
                  <option value="">Label</option>
                </select>
              </b-col>
            </b-row>

            <b-row class="basicInformation m-0" no-gutters>
              <b-col class="pt-1">
                <label for="alamatToko">Alamat Toko :</label>
              </b-col>
              <b-col>
                <!-- <b-form-input
                  id="alamatToko"
                  :style="{ textAlign: isViewMode ? 'right' : 'left' }"
                  size="md"
                  type="text"
                  v-model="alamat"
                  :plaintext="isViewMode"
                  :readonly="isViewMode"
                /> -->
                <b-form-input
                  id="alamatToko"
                  :style="{ textAlign: isViewMode ? 'right' : 'left' }"
                  size="md"
                  type="text"
                  v-model="alamat"
                  :plaintext="true"
                  :readonly="true"
                />
              </b-col>
            </b-row>

            <b-row
              class="basicInformation m-0"
              v-if="tanggalTransaksi"
              no-gutters
            >
              <b-col class="pt-1">
                <label for="tanggal">Tanggal Transaksi :</label>
              </b-col>
              <b-col class="text-right">
                {{ invDate }}
              </b-col>
            </b-row>
          </b-container>

          <div style="margin-top: 5px">
            <!-- <div>Daftar Pembelian:</div> -->
            <div class="separator" />
            <table class="itemListContainer table">
              <thead>
                <item-list-form
                  title
                  v-if="invoiceItems.length > 0"
                  :mode="viewMode"
                />
              </thead>
              <tbody>
                <item-list-form
                  v-for="(item, index) in invoiceItems"
                  :key="`${index}`"
                  ref="items"
                  :mode="viewMode"
                  :item="item"
                  :hapus="
                    () => {
                      invoiceItems.splice(index, 1);
                    }
                  "
                />
              </tbody>
            </table>
          </div>

          <div v-if="returItems.length > 0">
            <div>Barang Retur:</div>
            <div class="separator" />
            <table class="itemListContainer table">
              <thead>
                <item-list-form title :mode="viewMode" />
              </thead>
              <tbody>
                <item-list-form
                  v-for="(item, index) in returItems"
                  :key="`${index}`"
                  ref="items"
                  :mode="viewMode"
                  :item="item"
                  :hapus="
                    () => {
                      returItems.splice(index, 1);
                    }
                  "
                />
              </tbody>
            </table>
          </div>

          <div
            class="d-flex justify-content-end row"
            style="width: 100%"
            v-if="!isViewMode"
          >
            <b-button
              class="text-decoration-none text-tombol"
              variant="link"
              @click="
                () => {
                  showModal(true);
                }
              "
              >+ Tambah Barang</b-button
            >
            <b-button
              class="text-decoration-none text-retur"
              variant="link"
              @click="
                () => {
                  showModal(false);
                }
              "
              >+ Tambah Retur</b-button
            >

            <br />
          </div>

          <div class="d-flex" style="page-break-before: auto;">
            <div class="d-flex flex-column p-2" style="width: 350px; margin-top: 5px">
              <div class="font-weight-bold text-center mb-auto">
                <label>Tanda Tangan</label>
              </div>
              <div class="text-center">
                <label>
                  (...............................................................................)
                </label>
              </div>

            </div>

            <div class="ml-auto p-2" style="width: 350px; margin-top: 5px">
              <div class="inlineKeyVal">
                <label>Total Pembelian :</label>
                <div>{{ currencyFn(total_pembelian) }}</div>
              </div>

              <div class="inlineKeyVal" style="padding-right: 10px">
                <label>Diskon :</label>
                <div class="row align-items-center">
                  <b-form-input
                    class="text-right"
                    :plaintext="isViewMode"
                    :readonly="isViewMode"
                    size="md"
                    style="width: 50px; "
                    v-model.number="diskon"
                  />
                  %
                </div>
              </div>
<!-- 
              <div
                class="d-flex justify-content-end"
                style="text-decoration: line-through"
              >
                {{ currencyFn(potongan_diskon) }}
              </div> -->

              <div class="inlineKeyVal">
                <label>Potongan Harga :</label>
                <label style="font-weight: bold">{{ currencyFn(potongan) }}</label>
              </div>

              <div class="inlineKeyVal">
                <label style="font-weight: bolder; font-size: 24px"
                  >Jumlah Akhir</label
                >
                <label style="font-weight: bolder; font-size: 24px">{{
                  total_akhir
                }}</label>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div
        class="d-flex justify-content-center"
        :style="{ padding: '0 30px 30px 30px' }"
      >
        <b-button class="tombol btn btn-danger" @click="batal">{{
          isViewMode ? "Keluar" : "Batal"
        }}</b-button>
        <b-button
          class="tombol btn btn-success"
          v-print="'printPage'"
          v-if="isViewMode"
          >Print</b-button
        >
        <b-button class="tombol btn btn-success" @click="actionFn" v-else>{{
          actionCaption
        }}</b-button>
      </div>
    </b-overlay>
  </div>
</template>

<script>
import { ItemListForm } from "../components";
import { Store } from "../store";
import { BarangService, TransaksiService, PembeliService } from "../helpers/servicesAPI";
import Print from "vue-print-nb";
import { currencyFormatting } from "../helpers/common";
import 'select2';
import $ from 'jquery';

const tabImage = require("../assets/17545.jpg");

export default {
  directives: {
    Print,
  },
  components: { ItemListForm },
  props: {
    idInvoice: [Number, String],
    viewMode: {
      type: String,
    },
  },
  data: () => ({
    tabImage: tabImage,
    uploading: false,

    namaPembeli: "",
    alamat: "",
    diskon: 0,
    tanggalTransaksi: null,

    invItem: true,
    tmpItem: {
      namaBarang: "",
      stock: null,
      harga: null,
    },

    invoiceItems: [],
    returItems: [],
    items: [],
    errorMessage: "",
    currencyFn: currencyFormatting,
    optionsPembeli: [],
    idPembeli: null
  }),
  computed: {
    invDate() {
      return `${this.tanggalTransaksi.getFullYear()}-${(
        this.tanggalTransaksi.getMonth() + 1
      )
        .toString()
        .padStart(
          2,
          "0"
        )}-${this.tanggalTransaksi.getDate().toString().padStart(2, "0")}`;
    },
    isViewMode() {
      return this.viewMode === "VIEW";
    },
    itemHelper() {
      return this.items.length > 0 ? this.items.map((i) => i.namaBarang) : [];
    },
    total_pembelian() {
      return this.invoiceItems.length > 0
        ? this.invoiceItems.reduce(
            (total, item) => total + item.harga * item.stock,
            0
          )
        : 0;
    },
    potongan() {
      return this.returItems.length > 0
        ? this.returItems.reduce(
            (total, item) => total + item.harga * item.stock,
            0
          )
        : 0;
    },
    potongan_diskon() {
      return this.diskon > 0
        ? Math.round((this.total_pembelian * this.diskon) / 100)
        : 0;
    },
    total_akhir() {
      return currencyFormatting(
        this.total_pembelian - this.potongan_diskon - this.potongan
      );
    },
    actionCaption() {
      switch (this.viewMode) {
        case "VIEW":
          return "Print";
        case "EDIT":
          return "Update Invoice";
        case "CREATE":
          return "Buat Invoice";
        default:
          return "UNKNOWN";
      }
    },
  },
  methods: {
    validateInvoice() {
      return !!this.namaPembeli && !!this.alamat;
    },
    showModal(isInvItem) {
      this.invItem = isInvItem;

      this.tmpItem = {
        namaBarang: "",
        stock: null,
        harga: null,
      };
      this.$bvModal.show("item-modal");
    },
    addInvoiceItem() {
      const itemIndex = this.itemHelper.indexOf(this.tmpItem.namaBarang);
      if (itemIndex < 0) {
        this.errorMessage = "Barang tidak tercatat di Database";
        this.$bvModal.show("error-modal");
        return;
      }

      const itemData = this.items[itemIndex];
      if(itemData.stockBarang - this.tmpItem.stock < 0) {
        this.errorMessage = itemData.namaBarang + " hanya tersisa " + itemData.stockBarang;
        this.$bvModal.show("error-modal");
        return;
      }

      if (this.invItem) {
        this.invoiceItems.push({ ...this.tmpItem });
      } else {
        this.returItems.push({ ...this.tmpItem });
      }

      this.$bvModal.hide("item-modal");
    },
    buatInvoice() {
      if (this.validateInvoice()) {
        this.uploading = true;

        for(const item of this.invoiceItems) {
            const itemIndex = this.itemHelper.indexOf(item.namaBarang);
            if (itemIndex < 0) {
              this.errorMessage = "Barang tidak tercatat di Database";
              this.$bvModal.show("error-modal");
              return;
            }

            const itemData = this.items[itemIndex];
            if(itemData.stockBarang - item.stock < 0) {
              this.errorMessage = itemData.namaBarang + " hanya tersisa " + itemData.stockBarang;
              this.$bvModal.show("error-modal");
              return;
            }
        }

        alert('berhasil');

        TransaksiService.buatInvoice({
          namaPembeli: this.namaPembeli,
          alamat: this.alamat,
          diskon: this.diskon,
          listBarangJual: this.invoiceItems,
          listBarangRetur: this.returItems,
          DP: 0,
        })
          .then(() => {
            this.$router.back();
          })
          .catch(() => {
            this.uploading = false;
            this.errorMessage = "Terjadi kesalahan saat membuat invoice";
            this.$bvModal.show("error-modal");
          });
      } else {
        this.errorMessage = "Harap isi semua informasi yang dibutuhkan";
        this.$bvModal.show("error-modal");
      }
    },
    updateInvoice() {
      if (this.validateInvoice()) {
        this.uploading = true;

        TransaksiService.editInvoice(this.idInvoice, {
          idTransaksi: this.idInvoice,
          namaPembeli: this.namaPembeli,
          alamat: this.alamat,
          diskon: this.diskon,
          listBarangJual: this.invoiceItems,
          listBarangRetur: this.returItems,
          nominalTransaksi: 0,
          hutangTransaksi: 0,
        })
          .then(() => {
            this.$router.back();
          })
          .catch(() => {
            this.uploading = false;
            this.errorMessage = "Terjadi kesalahan saat mengupdate invoice";
            this.$bvModal.show("error-modal");
          });
      } else {
        this.errorMessage = "Harap isi semua informasi yang dibutuhkan";
        this.$bvModal.show("error-modal");
      }
    },
    actionFn() {
      switch (this.viewMode) {
        case "EDIT": {
          this.updateInvoice();
          break;
        }
        case "CREATE": {
          this.buatInvoice();
          break;
        }
      }
    },
    batal() {
      this.$router.back();
    },
    requestBarang() {
      BarangService.tampilkanItem()
        .then((response) => {
          if (response && response.status === 200) {
            this.baseBarang = response.result;
            Store.commit("setListBarang", response.result);
          }
        })
        .finally(() => {
          this.itemReady = true;
        });
    },
    viewInvoice() {
      this.uploading = true;
      // console.log("id = " + this.idInvoice);
      TransaksiService.viewInvoice(this.idInvoice)
        .then((response) => {
          // console.log("response = " + response);
          if (response && response.status === 200) {
            const data = response.result;
            // console.log("data = " + data);

            this.namaPembeli = data.namaPembeli;
            this.alamat = data.alamat;
            this.diskon = data.diskon;
            this.tanggalTransaksi = new Date(data.tanggalTransaksi);
            data.listBarangJual.forEach((l) => {
              this.invoiceItems.push({
                namaBarang: l.barangModel.namaBarang,
                stock: l.stockBarangJual,
                harga: l.hargaJual,
              });
            });
            data.listBarangRetur.forEach((l) => {
              this.returItems.push({
                namaBarang: l.barangModel.namaBarang,
                stock: l.stockBarangRetur,
                harga: l.hargaRetur,
              });
            });
          }
        })
        .catch(() => {
          // console.log(e.message);
          this.errorMessage = "Data invoice tidak ditemukan";
          this.$bvModal.show("error-modal");
        })
        .finally(() => {
          this.uploading = false;
        });
    },
    getListPembeli(isChange) {
      PembeliService.tampilkanListPembeli()
          .then(response => {
            const data = response.data;
            data.forEach((x) => {
              this.optionsPembeli.push({
                value: x.idPembeli,
                text: x.namaPembeli + " ("+ x.alamat+")"
              });
              

              var namaPembeliOpt = new Option(x.namaPembeli + " ("+ x.alamat+")", x.idPembeli, true, true)
              if(isChange) {
                $('#namaToko').append(namaPembeliOpt).trigger('change');
                if(x.namaPembeli == this.namaPembeli && x.alamat == this.alamat)
                  $('#namaToko').val(x.idPembeli).trigger('change');
              } else {
                $('#namaToko').append(namaPembeliOpt);
                if(x.namaPembeli == this.namaPembeli && x.alamat == this.alamat) 
                  $('#namaToko').val(x.idPembeli).trigger('change');
              }
            })
          })
          .catch(() => {
            this.optionsPembeli = [];
          })
      return this.optionsPembeli;
    },
    getAlamat(id_pembeli) {
      PembeliService.viewPembeli(id_pembeli)
          .then(response => {
            this.namaPembeli = response.namaPembeli
            this.alamat = response.alamat
          })
          .catch(() => {
            this.alamat = ""
          })
      return this.alamat;
    }
  },
  created() {
    if (this.idInvoice && this.viewMode !== "CREATE") {
      this.viewInvoice();
    }

    if (this.viewMode !== "VIEW") {
      const storedList = Store.getters.listBarang;
      if (storedList && storedList.length > 0) {
        this.items = storedList;
      }

      this.requestBarang();
    }
  },
  mounted() {
    if (this.viewMode !== "VIEW")
      this.getListPembeli(false)
    const self = this;
    if(!this.isViewMode) {
      $('#namaToko').select2({
        placeholder: 'Pilih Toko',
        width: '100%',
        data: this.optionsPembeli
      })
      $('#namaToko').on('change', function() {
        this.idPembeli = $('#namaToko').val();
        self.getAlamat(this.idPembeli);
      })
    }
  }
};
</script>

<style scoped>
@media print {
  /* #merk{
    display: flex!important;;
  } */
  #content-area {
    padding: 0 30px !important;
  }
  @page {
    size: auto;
    width: 11in;
    height: 9.5in;
  }
}
#merk {
  display: none;
}
#content-area {
  padding: 30px;
}
.basicInformation {
  margin: 10px 0;
}
.inlineKeyVal {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 0;
}
.text-tombol {
  font-weight: bolder;
  font-size: 18px;
  color: deepskyblue;
}
.text-retur {
  font-weight: bolder;
  font-size: 18px;
  color: darkblue;
}
.tombol {
  min-width: 120px;
  margin: 0 15px;
}
.separator {
  margin-top: 5px;
  width: 100%;
  height: 1px;
  background-color: black;
}

p {
  font-size: 25px;
}

#printPage {
  font-size: 18px;
}

#printPage input {
  font-size: 18px;
}
</style>