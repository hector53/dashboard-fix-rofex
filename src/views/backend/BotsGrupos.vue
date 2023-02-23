<script setup>
import { reactive, onMounted, watch, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useTemplateStore } from "@/stores/template";
// Template components
import widgetBot from "@/components/botGrupos/widgetBot.vue";
import axios from "axios";
import Swal from "sweetalert2";
const route = useRoute();
const router = useRouter();
const id = computed(() => route.params.id);
const data = reactive({
  arrayGrupos: [],
  nameGrupo: "",
  arrayBots: [],
  arrayBotsList: [],
  showModalBot: false,
});
const store = useTemplateStore();
// Set default properties
let toast = Swal.mixin({
  buttonsStyling: false,
  target: "#page-container",
  customClass: {
    confirmButton: "btn btn-success m-1",
    cancelButton: "btn btn-danger m-1",
    input: "form-control",
  },
});

async function cambiarGrupo(id) {
  router.push({ name: "backend-grupos-id", params: { id: id } });
}

watch(id, (newValue, oldValue) => {
  console.log(`El valor de ID ha cambiado de ${oldValue} a ${newValue}`);
  get_bots_by_grupo(newValue);
});

async function get_bots_by_grupo(id) {
  await axios
    .get(store.urlBackend + "/get_bots_by_grupo/" + id)
    .then((response) => {
      console.log("bots grupo", response);
      data.arrayBots = response.data;
    });
}

async function eliminarGrupo(id) {
  toast
    .fire({
      title: "Estas seguro?",
      text: "perderas todo esto!",
      icon: "warning",
      showCancelButton: true,
      customClass: {
        confirmButton: "btn btn-danger m-1",
        cancelButton: "btn btn-secondary m-1",
      },
      confirmButtonText: "Si!",
      html: false,
      preConfirm: () => {
        return new Promise((resolve) => {
          setTimeout(() => {
            resolve();
          }, 50);
        });
      },
    })
    .then((result) => {
      if (result.value) {
        deleteGrupo(id);
        // result.dismiss can be 'overlay', 'cancel', 'close', 'esc', 'timer'
      } else if (result.dismiss === "cancel") {
        toast.fire("Cancelled", "El grupo no ha sido borrado", "error");
      }
    });
}

async function deleteGrupo(id_bot) {
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/delete_grupo/" + id_bot)
    .then((response) => {
      console.log(response);
      get_grupos();
      if (id.value) {
        get_bots_by_grupo(id.value);
      }
      store.pageLoader({ mode: "off" });
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

async function addGrupo() {
  store.pageLoader({ mode: "on" });
  await axios
    .post(store.urlBackend + "/add_grupo", {
      name: data.nameGrupo,
    })
    .then((response) => {
      console.log(response);
      get_grupos();
      store.pageLoader({ mode: "off" });
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

async function get_bots_list_by_grupo(id) {
  await axios
    .get(store.urlBackend + "/get_bots_list_by_grupo/" + id)
    .then((response) => {
      console.log("bots grupo", response);
      data.arrayBotsList = response.data;
    });
}

async function get_grupos() {
  await axios.get(store.urlBackend + "/get_grupos").then((response) => {
    console.log("grupos", response);
    data.arrayGrupos = response.data.grupos;
  });
}

async function addBotToGrupo(id_bot) {
  console.log("agregar bot al grupo");
  store.pageLoader({ mode: "on" });
  await axios
    .post(store.urlBackend + "/add_bot_to_grupo", {
      id_bot: id_bot,
      id_grupo: id.value,
    })
    .then((response) => {
      console.log(response);
      get_bots_by_grupo(id.value);
      store.pageLoader({ mode: "off" });
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

function eliminarBotDelGrupo(id_bot) {
  console.log("Datos recibidos en el componente padre:", id_bot);
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/delete_bot_from_grupo", {
      id_bot: id_bot,
      id_grupo: id.value,
    })
    .then((response) => {
      console.log(response);
      get_bots_by_grupo(id.value);
      store.pageLoader({ mode: "off" });
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

onMounted(async () => {
  await get_grupos();
  if (id.value) {
    get_bots_by_grupo(id.value);
  }
  var ModalAddBot = document.getElementById("modal-add-bot");
  ModalAddBot.addEventListener("hidden.bs.modal", function (event) {
    console.log("hidden modal bot add");
    // do something...
  });
  ModalAddBot.addEventListener("show.bs.modal", function (event) {
    // do something...
    console.log("show modal bot add");
    if (id.value) {
      get_bots_list_by_grupo(id.value);
    }
  });
});
</script>

<template>
  <!-- Modal ADD GRUPO -->
  <div
    class="modal fade"
    id="modal-add-grupo"
    tabindex="-1"
    role="dialog"
    aria-labelledby="modal-add-grupo"
    aria-hidden="true"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <BaseBlock title="Add Grupo" transparent class="mb-0">
          <template #options>
            <button
              type="button"
              class="btn-block-option"
              data-bs-dismiss="modal"
              aria-label="Close"
            >
              <i class="fa fa-fw fa-times"></i>
            </button>
          </template>

          <template #content>
            <div class="block-content block-content-full text-end bg-body">
              <div class="row">
                <div class="col-lg-12 space-y-5">
                  <!-- Form Labels on top - Default Style -->
                  <form @sumbit.prevent>
                    <div class="mb-4">
                      <input
                        type="text"
                        class="form-control"
                        placeholder="Nombre de grupo"
                        v-model="data.nameGrupo"
                      />
                    </div>
                  </form>
                </div>
              </div>
              <button
                type="button"
                class="btn btn-sm btn-alt-secondary me-1"
                data-bs-dismiss="modal"
              >
                Close
              </button>
              <button
                type="button"
                class="btn btn-sm btn-primary"
                data-bs-dismiss="modal"
                @click="addGrupo()"
              >
                Add
              </button>
            </div>
          </template>
        </BaseBlock>
      </div>
    </div>
  </div>
  <!-- END Modal ADD GRUPO -->

  <!-- Modal ADD bot -->
  <div class="modal fade" id="modal-add-bot" tabindex="-1" role="dialog">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <BaseBlock title="Add bot" transparent class="mb-0">
          <template #options>
            <button
              type="button"
              class="btn-block-option"
              data-bs-dismiss="modal"
              aria-label="Close"
            >
              <i class="fa fa-fw fa-times"></i>
            </button>
          </template>

          <template #content>
            <div class="block-content block-content-full text-end bg-body">
              <div class="list-group push">
                <a
                  v-for="(item, index) in data.arrayBotsList"
                  :key="index"
                  class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
                  href="javascript:void(0)"
                  @click="addBotToGrupo(item.id)"
                  data-bs-dismiss="modal"
                >
                  <span>{{ item.type }}</span> <span>{{ item.symbols }}</span>
                </a>
              </div>

              <button
                type="button"
                class="btn btn-sm btn-alt-secondary me-1"
                data-bs-dismiss="modal"
              >
                Close
              </button>
            </div>
          </template>
        </BaseBlock>
      </div>
    </div>
  </div>
  <!-- END Modal ADD GRUPO -->

  <!-- Page Content -->
  <div class="content">
    <BaseBlock
      title="Grupos"
      ref="blockGrupos"
      transparent
      :header-bg="false"
      btn-option-content
    >
      <template #options>
        <button
          type="button"
          class="btn-block-option"
          data-bs-toggle="modal"
          data-bs-target="#modal-add-grupo"
        >
          <i class="si si-plus"></i>
        </button>
      </template>

      <div class="row text-center">
        <div
          class="col-sm-3"
          v-for="(item, index) in data.arrayGrupos"
          :key="index"
        >
          <div
            class="alert alert-dismissible"
            :class="{
              'alert-light': item.id == id,
              'alert-secondary': item.id != id,
            }"
            role="alert"
          >
            <a
              style="cursor: pointer"
              class="alert-heading h5 my-2"
              @click="cambiarGrupo(item.id)"
            >
              {{ item.name }}
            </a>

            <button
              type="button"
              class="btn-close"
              aria-label="Close"
              @click="eliminarGrupo(item.id)"
            ></button>
          </div>
        </div>
      </div>
    </BaseBlock>

    <BaseBlock
      title="Bots"
      ref="blockBots"
      transparent
      :header-bg="false"
      btn-option-content
    >
      <template #options>
        <button
          type="button"
          class="btn-block-option"
          data-bs-toggle="modal"
          data-bs-target="#modal-add-bot"
        >
          <i class="si si-plus"></i>
        </button>
      </template>

      <div class="row">
        <div
          class="col-sm-6"
          v-for="(item, index) in data.arrayBots"
          :key="index"
        >
          <widget-bot
            :ordenesBot="item"
            @eliminar-bot-del-grupo="eliminarBotDelGrupo"
          ></widget-bot>
        </div>
      </div>
    </BaseBlock>
  </div>
</template>
<style>
.btnGrupo {
  background: #253041;
  cursor: pointer;
}
.btnGrupo.active {
  background: #253b5d;
}
</style>
