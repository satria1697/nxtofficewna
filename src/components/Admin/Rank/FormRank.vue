<template>
  <transition class="modal" tabindex="-1" role="dialog">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-dialog modal-dialog-centered container-md">
          <div class="modal-content">
            <div class="modal-header bg-default">
              <span class="font-weight-bold">{{ title }}</span>
              <i
                class="fa fa-window-close pull-right pointer-event"
                aria-hidden="true"
                @click="closeModal()"
              ></i>
            </div>
            <div class="modal-body">
              <user-modal
                v-if="isUserModal"
                :title="textTitle"
                :textSuccess="berhasil"
                :textDanger="!berhasil"
                @modal-closed="closeModal"
              />
              <delete-modal
                :data="dataAll"
                v-if="isDeleteModal"
                @modal-closed="isDeleteModal = false"
                @delete-data="deleteData"
              />
              <form class="container-fluid">
                <div class="row">
                  <div v-if="editId === null" class="form form-group col-4">
                    <label for="formID" class="top">ID</label>
                    <input
                      id="formID"
                      class="bottom form-control"
                      v-model="dataAll.id"
                    />
                  </div>
                  <div v-else class="form form-group col-4">
                    <label for="formID" class="top top-disabled">ID</label>
                    <input
                      id="formID"
                      class="bottom form-control disabled"
                      v-model="dataAll.id"
                      disabled
                    />
                  </div>
                </div>
                <div class="row">
                  <div class="form form-group col">
                    <label for="formDesc" class="top">Description</label>
                    <input
                      id="formDesc"
                      class="bottom form-control"
                      v-model="dataAll.description"
                    />
                  </div>
                </div>
              </form>
            </div>
            <div v-if="editId === null" class="modal-footer">
              <button class="btn btn-default" v-on:click="reset()">
                <i class="fas fa-eraser"></i> Reset
              </button>
              <button
                class="btn btn-default"
                v-on:click="register('submit', null)"
              >
                <i class="fas fa-save"></i> Simpan
              </button>
            </div>
            <div v-if="editId !== null" class="modal-footer">
              <button class="btn btn-default" v-on:click="isDeleteModal = true">
                <i class="fas fa-trash"></i> Delete
              </button>
              <button
                class="btn btn-default"
                v-on:click="regiser('update', dataAll.id)"
              >
                <i class="fas fa-save"></i>
                Simpan Perubahan
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import Api from "../../../api";

export default {
  props: {
    editId: {
      type: Number
    },
    title: {
      type: String
    }
  },
  data() {
    return {
      dataAll: {
        id: null,
        description: ""
      },
      berhasil: true,
      uploaded: false,
      updated: false,
      deleted: false,
      isDeleteModal: false,
      textTitle: "",
      isUserModal: false
    };
  },
  created() {
    const escapeHandler = e => {
      // console.log(e);
      if (e.key === "Escape") {
        this.closeModal();
      }
    };
    document.addEventListener("keydown", escapeHandler);
    this.$once("hook:destroyed", () => {
      document.removeEventListener("keydown", escapeHandler);
    });
  },
  mounted() {
    this.checkEdit();
  },
  methods: {
    closeModal() {
      this.reset();
      this.$emit("get-data");
      this.$emit("modal-closed");
    },
    reset() {
      this.dataAll.id = null;
      this.dataAll.description = "";
      // this.editId = null;
    },
    checkEdit() {
      if (this.editId !== null) {
        Api.rank
          .find(this.editId)
          .then(resp => {
            this.dataAll = resp.data.data;
          })
          .catch(error => {
            console.log(error);
            this.reset();
          });
      }
    },
    register(setup, id) {
      let rawData = {
        id: this.dataAll.id,
        description: this.dataAll.description
      };
      let formData = new FormData();
      for (let key in rawData) {
        formData.append(key, rawData[key]);
      }
      if (setup === "submit") {
        Api.rank
          .register(formData)
          .then(resp => {
            if (resp.data.status === "success") {
              this.textTitle = "Data berhasil disimpan";
              this.berhasil = true;
              this.isUserModal = true;
            } else {
              this.textTitle =
                "Input belum masuk kedatabase, silahkan coba lagi";
              this.berhasil = false;
              this.isUserModal = true;
            }
          })
          .catch(err => {
            console.log(err);
            this.textTitle = "Input data salah, silahkan cek kembali";
            this.berhasil = false;
            this.isUserModal = true;
          });
      } else {
        Api.rank
          .update(id, formData)
          .then(resp => {
            if (resp.data.status === "success") {
              this.textTitle = "Data berhasil diperbaharui";
              this.berhasil = true;
              this.isUserModal = true;
            } else {
              this.textTitle =
                "Input belum masuk kedatabase, silahkan coba lagi";
              this.berhasil = false;
              this.isUserModal = true;
            }
          })
          .catch(err => {
            this.textTitle = "Input data salah, silahkan cek kembali";
            this.berhasil = false;
            this.isUserModal = true;
            console.log(err);
          });
      }
    },
    deleteData(data) {
      console.log(data);

      Api.rank
        .delete(data.id)
        .then(resp => {
          if (resp.status === 204) {
            this.textTitle = "Data berhasil dihapus";
            this.berhasil = true;
            this.isUserModal = true;
          } else {
            this.textTitle = "Data gagal dihapus";
            this.berhasil = false;
            this.isUserModal = true;
          }
        })
        .catch(err => {
          this.textTitle = "Data gagal dihapus";
          this.berhasil = false;
          this.isUserModal = true;
          console.log(err);
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.pull-right {
  margin: 8px 10px 0 0;
}
</style>
