<script setup>
import { reactive, computed, onMounted, nextTick } from "vue";
import useVuelidate from "@vuelidate/core";
import { required, minLength } from "@vuelidate/validators";
import { useTemplateStore } from "@/stores/template";
import axios from "axios";
import Swal from "sweetalert2";
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

//variables para usar en el componente

const data = reactive({
  isSessionActive: false,
  isSecuritysOn: false,
  sessionID: null,
  arrayBots: [],
  searchSymbol: "",
  symbolsSelected: [],
  isEditBot: false,
});
const dataEdit = reactive({
  idBot: null,
  symbolsBot: null,
  typeBot: null,
  statusBot: null,
});

/*
watch(data, async (currentValue, oldvalue) => {
  if (currentValue.sessionID > 0 && currentValue.arrayBots.length == 0) {
    console.log("si hay id de session", currentValue.sessionID);
    await get_bots();
  }
});
*/
// Main store
const store = useTemplateStore();

const optionsLiveDemo = reactive([
  { value: null, text: "Select Live or Demo" },
  { value: 1, text: "Live" },
  { value: 2, text: "Demo" },
]);
// Input state variables for the form fix
const state = reactive({
  target: "ROFX",
  sender: null,
  password: null,
  account: null,
  liveDemo: null,
});

//inputs para el formulario de bots
const inputsBots = reactive({
  type_bot: null,
  minRate: null,
  maxRate: null,
  spreadMin: "",
  sizeMax: null,
});
// Validation rules
const rules = computed(() => {
  return {
    target: {
      required,
      minLength: minLength(3),
    },
    sender: {
      required,
      minLength: minLength(3),
    },
    password: {
      required,
      minLength: minLength(5),
    },

    account: {
      required,
      minLength: minLength(3),
    },
    liveDemo: {
      required,
    },
  };
});
// Use vuelidate
const v$ = useVuelidate(rules, state);
// On form submission

async function get_securitys(t = 0) {
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/get_securitys", {
      id_fix: data.sessionID,
    })
    .then((response) => {
      console.log(response);
      store.pageLoader({ mode: "off" });
      if (t == 0) {
        toast.fire("Success", "Everything was updated perfectly!", "success");
      }
      t;
      if (response.data.status == true) {
        data.isSecuritysOn = true;
        if (state.liveDemo == 2) {
          localStorage.securityFixDemo = JSON.stringify(response.data);
        } else {
          localStorage.securityFixLive = JSON.stringify(response.data);
        }
      }
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

async function iniciar_fix() {
  // perform async actions
  store.pageLoader({ mode: "on" });
  await axios
    .post(store.urlBackend + "/iniciar_fix", state)
    .then((response) => {
      console.log(response);
      store.pageLoader({ mode: "off" });
      get_session_fix();
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", error.response.data.message, "error");
    });
}

async function detener_fix() {
  // perform async actions
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/detener_fix", {
      id_fix: data.sessionID,
    })
    .then((response) => {
      console.log(response);
      store.pageLoader({ mode: "off" });
      state.target = "";
      state.sender = "";
      state.password = "";
      state.account = "";
      state.liveDemo = "";
      //quiero que se marque como selected el valor que viene de la base de datos

      data.isSessionActive = false;
      data.isSecuritysOn = false;
      data.sessionID = null;
      toast.fire("Success", "Everything was updated perfectly!", "success");
      localStorage.removeItem("securityFix");
      localStorage.removeItem("sesionFix");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

async function get_bots() {
  console.log("entrando a get_bots");
  await axios
    .get(store.urlBackend + "/get_bots/" + data.sessionID)
    .then((response) => {
      console.log("bots", response);
      data.arrayBots = response.data.arrayBots;
    });
}

async function get_session_fix() {
  // perform async actions
  store.pageLoader({ mode: "on" });
  await axios
    .get(store.urlBackend + "/get_session_fix", state)
    .then((response) => {
      console.log(response);
      if (response.data.status == true) {
        state.target = response.data.data.target;
        state.sender = response.data.data.sender;
        state.password = response.data.data.password;
        state.account = response.data.data.account;
        state.liveDemo = response.data.data.liveDemo;
        //quiero que se marque como selected el valor que viene de la base de datos

        data.isSessionActive = true;
        data.sessionID = response.data.data.id;
        if (localStorage.sesionFix) {
          localStorage.sesionFix = response.data.data.id;
        } else {
          localStorage.sesionFix = response.data.data.id;
        }

        nextTick(() => {
          console.log("nextTick", data);
        });
      }
      store.pageLoader({ mode: "off" });
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

async function onSubmit() {
  const result = await v$.value.$validate();

  if (!result) {
    // notify user form is invalid
    return;
  }
  if (data.isSessionActive) {
    await detener_fix();
  } else {
    await iniciar_fix();
  }
}

const filteredSymbols = computed(() => {
  if (data.searchSymbol != "") {
    var parseSecuritys = JSON.parse(localStorage.securityFixDemo);
    if (state.liveDemo == 1) {
      parseSecuritys = JSON.parse(localStorage.securityFixLive);
    }
    var arraySymbols = JSON.parse(parseSecuritys.securitys);
    var arrayOnlySymbols = arraySymbols.map((item) => item.symbol) || [];
    var newArray = arrayOnlySymbols.filter((option) =>
      option.toLowerCase().includes(data.searchSymbol.toLowerCase())
    );
    return newArray;
  } else {
    return [];
  }
});

function editBot(id, symbols, type, status, opciones) {
  console.log("editBot", id, symbols, type, status);
  data.isEditBot = true;
  dataEdit.idBot = id;
  dataEdit.symbolsBot = symbols;
  inputsBots.type_bot = type;
  inputsBots.sizeMax = opciones.sizeMax;
  if (type == 1) {
    inputsBots.minRate = opciones.minRate;
    inputsBots.maxRate = opciones.maxRate;
  }
  if (type == 0) {
    inputsBots.spreadMin = opciones.spreadMin;
  }
}

function detenerBot(id) {
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/detenerBot", {
      id: id,
      id_fix: data.sessionID,
    })
    .then((response) => {
      console.log(response);
      get_bots();
      store.pageLoader({ mode: "off" });
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

function startBot(id) {
  if (data.isSessionActive && data.isSecuritysOn) {
    store.pageLoader({ mode: "on" });
    axios
      .post(store.urlBackend + "/startBot", {
        id: id,
        id_fix: data.sessionID,
      })
      .then((response) => {
        console.log(response);
        get_bots();
        store.pageLoader({ mode: "off" });
        toast.fire("Success", "Everything was updated perfectly!", "success");
      })
      .catch((error) => {
        console.log(error);
        store.pageLoader({ mode: "off" });
        toast.fire("Oops...", "Something went wrong!", "error");
      });
  }
}

function deleteBot(id) {
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/deleteBot", {
      id: id,
    })
    .then((response) => {
      console.log(response);
      get_bots();
      store.pageLoader({ mode: "off" });
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

function editBotToDB(id) {
  var id_fix = 0;
  var opciones = {};
  if (data.isSessionActive) {
    id_fix = data.sessionID;
  }
  if (inputsBots.type_bot == 1) {
    opciones = {
      minRate: inputsBots.minRate,
      maxRate: inputsBots.maxRate,
      sizeMax: inputsBots.sizeMax,
    };
  }
  if (inputsBots.type_bot == 0) {
    opciones = {
      spreadMin: inputsBots.spreadMin,
      sizeMax: inputsBots.sizeMax,
    };
  }
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/editBot", {
      id_bot: id,
      id_fix: id_fix,
      opciones: opciones,
      typeBot: inputsBots.type_bot,
    })
    .then((response) => {
      console.log(response);
      data.isEditBot = false;
      get_bots();
      store.pageLoader({ mode: "off" });
      toast.fire("Success", "Everything was updated perfectly!", "success");
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
      toast.fire("Oops...", "Something went wrong!", "error");
    });
}

function addBotToDB() {
  if (data.symbolsSelected.length > 0) {
    store.pageLoader({ mode: "on" });
    axios
      .post(store.urlBackend + "/add_bot", {
        symbols: data.symbolsSelected,
        opciones: inputsBots,
      })
      .then((response) => {
        console.log(response);

        get_bots();
        store.pageLoader({ mode: "off" });
        toast.fire("Success", "Everything was updated perfectly!", "success");
      })
      .catch((error) => {
        console.log(error);
        store.pageLoader({ mode: "off" });
        toast.fire("Oops...", "Something went wrong!", "error");
      });
  }
}

function addSymbol(option) {
  data.symbolsSelected.push(option);
  data.searchSymbol = "";
}
function resetFormBot() {
  data.isEditBot = false;
  data.symbolsSelected = [];
  data.searchSymbol = "";
  inputsBots.maxRate = null;
  inputsBots.minRate = null;
  inputsBots.type_bot = null;
  inputsBots.sizeMax = null;
  inputsBots.spreadMin = "";
}

onMounted(async () => {
  console.log("Mounted");
  var ModalAddBot = document.getElementById("modal-bot");
  ModalAddBot.addEventListener("hidden.bs.modal", function (event) {
    console.log("hidden modal bot add");
    resetFormBot();

    // do something...
  });
  ModalAddBot.addEventListener("show.bs.modal", function (event) {
    // do something...
    console.log("show modal bot add");
  });
  // perform async actions

  await get_session_fix();
  await get_bots();
  //await get_bots();
  if (data.isSessionActive) {
    if (localStorage.securityFixDemo || localStorage.securityFixLive) {
      var securityFix = JSON.parse(localStorage.securityFixDemo);
      if (state.liveDemo == 1) {
        securityFix = JSON.parse(localStorage.securityFixLive);
      }
      // Obtener la fecha y hora actual
      const now = new Date();
      console.log(securityFix.date);
      // Crear la fecha y hora a partir de la cadena de texto (string)
      const fecha_str = securityFix.date;
      const fecha = new Date(fecha_str);
      // Comparar la fecha con la fecha y hora actual
      if (fecha.toDateString() === now.toDateString()) {
        console.log("La fecha pertenece al día actual.");
        await get_securitys(1);
      } else {
        console.log("La fecha no pertenece al día actual.");
        data.isSecuritysOn = false;
      }
    }
  }
});

const socket = new WebSocket("ws://localhost:5100");

socket.addEventListener("open", (event) => {
  socket.send("Hello Server!", event);
});

socket.addEventListener("message", (event) => {
  var data = JSON.parse(event.data);
  console.log("Message from server ", data);
  if (data.type == "security") {
    if (localStorage.securityFix) {
      var arraySymbols = JSON.parse(localStorage.securityFix);
      data.tickers.foreach((tick) => {
        if (!arraySymbols.includes(tick)) {
          arraySymbols.push(tick);
        }
      });
    } else {
      localStorage.securityFix = JSON.stringify(data.tickers);
    }
  }
});
</script>

<template>
  <!-- Fade In Block Modal -->
  <div
    class="modal fade"
    id="modal-bot"
    tabindex="-1"
    role="dialog"
    aria-labelledby="modal-bot"
    aria-hidden="true"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <BaseBlock title="Bot" transparent class="mb-0">
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
            <div class="block-content block-content-full text-left bg-body">
              <form @sumbit.prevent>
                <div class="row push">
                  <div class="col-lg-12">
                    <div class="mb-4">
                      <label class="form-check-label mb-2" for="type_bot"
                        >Type Bot</label
                      >
                      <select
                        :disabled="data.isEditBot"
                        class="form-select"
                        id="type_bot"
                        name="type_bot"
                        v-model="inputsBots.type_bot"
                      >
                        <option value="0">Triangulo</option>
                        <option value="1">CI-48</option>
                      </select>
                    </div>
                    <div class="mb-4">
                      <p v-if="inputsBots.type_bot == 0">
                        Bot tipo triangulo: necesita 3 simbolos, futuro1,
                        futuro2 y pase, deben ir en ese orden
                      </p>
                      <p v-if="inputsBots.type_bot == 1">
                        Bot tipo CI-48: necesita 2 simbolos, CI y 48, deben ir
                        en ese orden
                      </p>
                    </div>
                    <div class="mb-4" v-if="data.isEditBot == false">
                      <input
                        type="text"
                        class="form-control"
                        placeholder="Buscar simbolo"
                        v-model="data.searchSymbol"
                      />
                      <div
                        class="list-group push"
                        v-if="filteredSymbols.length"
                      >
                        <a
                          v-for="(option, index) in filteredSymbols"
                          :key="index"
                          class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
                          href="javascript:void(0)"
                          @click="addSymbol(option)"
                        >
                          {{ option }}
                        </a>
                      </div>
                    </div>

                    <div class="mb-4" v-if="data.symbolsSelected.length > 0">
                      <BaseBlock title="Symbols">
                        <div class="list-group push">
                          <a
                            v-for="(item, index) in data.symbolsSelected"
                            :key="index"
                            class="list-group-item list-group-item-action d-flex justify-content-between align-items-center"
                          >
                            {{ item }}
                            <i
                              class="far fa fa-circle-xmark"
                              style="color: red; cursor: pointer"
                              @click="data.symbolsSelected.splice(index, 1)"
                            ></i>
                          </a>
                        </div>
                      </BaseBlock>
                    </div>

                    <div class="mb-4" v-if="inputsBots.type_bot == 0">
                      <label class="form-check-label mb-2" for="spreadMin"
                        >Spread Min</label
                      >
                      <input
                        type="number"
                        class="form-control"
                        placeholder="Spread Min"
                        v-model="inputsBots.spreadMin"
                      />
                    </div>

                    <div class="mb-4" v-if="inputsBots.type_bot == 1">
                      <label class="form-check-label mb-2" for="type_bot"
                        >Minimum arbitrage rate</label
                      >
                      <input
                        type="number"
                        class="form-control"
                        placeholder="Min rate"
                        v-model="inputsBots.minRate"
                      />
                    </div>
                    <div class="mb-4" v-if="inputsBots.type_bot == 1">
                      <label class="form-check-label mb-2" for="type_bot"
                        >Maximum arbitrage rate</label
                      >
                      <input
                        type="number"
                        class="form-control"
                        placeholder="Max rate"
                        v-model="inputsBots.maxRate"
                      />
                    </div>
                    <div class="mb-4">
                      <label class="form-check-label mb-2" for="sizeMax"
                        >Size Max</label
                      >
                      <input
                        type="number"
                        class="form-control"
                        placeholder="Size Max"
                        v-model="inputsBots.sizeMax"
                      />
                    </div>
                  </div>
                </div>
              </form>

              <button
                type="button"
                class="btn btn-sm btn-alt-secondary me-1"
                data-bs-dismiss="modal"
              >
                Close
              </button>
              <button
                v-if="data.isEditBot == false"
                type="button"
                class="btn btn-sm btn-primary"
                data-bs-dismiss="modal"
                @click="addBotToDB()"
              >
                Add
              </button>
              <button
                v-else
                type="button"
                class="btn btn-sm btn-primary"
                data-bs-dismiss="modal"
                @click="editBotToDB(dataEdit.idBot)"
              >
                Edit
              </button>
            </div>
          </template>
        </BaseBlock>
      </div>
    </div>
  </div>
  <!-- END Fade In Block Modal -->
  <!-- Page Content -->
  <div class="content">
    <!-- Inline -->
    <BaseBlock title="Server Fix" content-full>
      <template #options>
        <div
          class="block-options-item text-success"
          v-if="data.isSessionActive"
        >
          ONLINE!
        </div>
        <div v-else class="block-options-item text-danger">OFFLINE!</div>

        <div class="block-options-item" v-if="data.isSecuritysOn">
          <span class="badge bg-success rounded-pill">Securitys ON</span>
        </div>

        <button
          type="button"
          :disabled="data.isSessionActive == false"
          @click="get_securitys()"
          v-else
          class="btn btn-sm btn-danger"
        >
          Securitys OFF
        </button>
      </template>
      <div class="row">
        <!-- Form Inline - Default Style -->
        <form
          class="row row-cols-lg-auto g-3 align-items-center"
          @submit.prevent="onSubmit"
        >
          <div class="col-12">
            <label class="visually-hidden" for="target">Target</label>
            <select
              :disabled="data.isSessionActive"
              class="form-select"
              id="target"
              name="target"
              placeholder="ROFX / BYMA"
              :class="{
                'is-invalid': v$.target.$errors.length,
              }"
              v-model="state.target"
              @blur="v$.target.$touch"
            >
              <option value="ROFX" selected>ROFX</option>
              <option value="BYMA">BYMA</option>
            </select>

            <div
              v-if="v$.target.$errors.length"
              class="invalid-feedback animated fadeIn"
            >
              Please enter a target
            </div>
          </div>
          <div class="col-12">
            <label class="visually-hidden" for="sender">Sender</label>
            <input
              :disabled="data.isSessionActive"
              type="text"
              class="form-control"
              id="sender"
              name="sender"
              placeholder="Username"
              :class="{
                'is-invalid': v$.sender.$errors.length,
              }"
              v-model="state.sender"
              @blur="v$.sender.$touch"
            />
            <div
              v-if="v$.sender.$errors.length"
              class="invalid-feedback animated fadeIn"
            >
              Please enter a username
            </div>
          </div>
          <div class="col-12">
            <label class="visually-hidden" for="password">Password</label>
            <input
              :disabled="data.isSessionActive"
              type="password"
              class="form-control"
              id="password"
              name="password"
              placeholder="Password"
              :class="{
                'is-invalid': v$.password.$errors.length,
              }"
              v-model="state.password"
              @blur="v$.password.$touch"
            />
            <div
              v-if="v$.password.$errors.length"
              class="invalid-feedback animated fadeIn"
            >
              Please provide a password
            </div>
          </div>
          <div class="col-12">
            <label class="visually-hidden" for="account">Account</label>
            <input
              :disabled="data.isSessionActive"
              type="text"
              class="form-control"
              id="account"
              name="account"
              placeholder="REM****"
              :class="{
                'is-invalid': v$.account.$errors.length,
              }"
              v-model="state.account"
              @blur="v$.account.$touch"
            />
            <div
              v-if="v$.sender.$errors.length"
              class="invalid-feedback animated fadeIn"
            >
              Please enter account
            </div>
          </div>

          <div class="col-12">
            <label class="visually-hidden" for="demo_real">Demo / Live</label>
            <select
              :disabled="data.isSessionActive"
              class="form-select"
              id="demo_real"
              name="demo_real"
              :class="{
                'is-invalid': v$.liveDemo.$errors.length,
              }"
              v-model="state.liveDemo"
              @blur="v$.liveDemo.$touch"
            >
              <option
                v-for="(option, index) in optionsLiveDemo"
                :value="option.value"
                :selected="option.value === state.liveDemo"
                :key="`option-${index}`"
              >
                {{ option.text }}
              </option>
            </select>
            <div
              v-if="v$.liveDemo.$errors.length"
              class="invalid-feedback animated fadeIn"
            >
              Please select a type Live or Demo!
            </div>
          </div>

          <div>
            <button type="submit" class="btn btn-primary">
              <div v-if="data.isSessionActive">
                <i class="fa fa-circle-stop"></i> Detener
              </div>
              <div v-else><i class="fa fa-circle-play"></i> Iniciar</div>
            </button>
          </div>
        </form>
        <!-- END Form Inline - Default Style -->
      </div>
    </BaseBlock>
    <!-- END Inline -->
    <!-- Partial Table -->
    <BaseBlock title="Bots">
      <template #options>
        <button
          type="button"
          class="btn-block-option"
          data-bs-toggle="modal"
          data-bs-target="#modal-bot"
          @click="data.isEditBot = false"
        >
          <i class="si si-plus"></i>
        </button>
      </template>
      <div class="btnEscucharMercados">
        <button
          type="button"
          @click="escuchar_mercados()"
          class="btn btn-sm btn-warning"
        >
          Escuchar Mercados <i class="fa fa-fw fa-ear-listen"></i>
        </button>
      </div>

      <table class="table table-bordered table-striped table-vcenter">
        <thead>
          <tr>
            <th class="fw-semibold fs-sm" style="width: 60%">Symbols</th>
            <th class="d-none d-md-table-cell" style="width: 10%">TypeBot</th>
            <th class="d-none d-md-table-cell" style="width: 10%">Status</th>
            <th class="text-center" style="width: 3%">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="bot in data.arrayBots" :key="bot.id">
            <td class="d-md-table-cell fs-sm">
              {{ bot.symbols }}
            </td>
            <td class="d-md-table-cell fs-sm">
              <span v-if="bot.type == 0">Triangulos</span>
              <span v-if="bot.type == 1">CI-48</span>
            </td>
            <td class="d-none d-sm-table-cell">
              <span class="badge rounded-pill bg-success" v-if="bot.status == 1"
                >Activo</span
              >
              <span class="badge rounded-pill bg-warning" v-else>Inactivo</span>
            </td>
            <td class="text-center">
              <div class="btn-group">
                <!--boton play o stop -->
                <button
                  type="button"
                  class="btn btn-sm btn-alt-secondary"
                  @click="startBot(bot.id)"
                  v-if="bot.status == 0"
                >
                  <i class="fa fa-play"></i>
                </button>
                <button
                  type="button"
                  class="btn btn-sm btn-alt-secondary"
                  @click="detenerBot(bot.id)"
                  v-else
                >
                  <i class="fa fa-stop"></i>
                </button>
                <!--boton escuchar o silenciar  -->
                <button
                  type="button"
                  class="btn btn-sm btn-alt-secondary"
                  v-if="bot.status == 1 || bot.status == 2"
                >
                  <i class="fa fa-volume-high"></i>
                </button>
                <button
                  type="button"
                  class="btn btn-sm btn-alt-secondary"
                  @click="escucharBotById(bot.id)"
                  v-else
                >
                  <i class="fa fa-volume-xmark"></i>
                </button>

                <button
                  type="button"
                  @click="
                    editBot(
                      bot.id,
                      bot.symbols,
                      bot.type,
                      bot.status,
                      bot.opciones
                    )
                  "
                  class="btn btn-sm btn-alt-secondary"
                  data-bs-toggle="modal"
                  data-bs-target="#modal-bot"
                >
                  <i class="fa fa-fw fa-edit"></i>
                </button>
                <button
                  type="button"
                  :disabled="bot.status == 0"
                  @click="dataBot(bot.id)"
                  class="btn btn-sm btn-alt-secondary"
                >
                  <i class="fa fa-fw fa-chart-pie"></i>
                </button>
                <button
                  type="button"
                  @click="deleteBot(bot.id)"
                  class="btn btn-sm btn-alt-secondary"
                >
                  <i class="fa fa-fw fa-times"></i>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </BaseBlock>
  </div>
</template>
<style>
.btnEscucharMercados {
  display: flex;
  margin-bottom: 20px;
  width: 100%;
  justify-content: flex-end;
}
</style>
