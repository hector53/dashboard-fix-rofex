<template>
  <div class="content">
    <div class="row">
      <div class="col-sm-12">
        <BaseBlock title="Bot Data" btn-option-content ref="blockBotData">
          <template #options>
            <button type="button" class="btn-block-option">
              <i class="si si-refresh"></i>
            </button>
          </template>
          <div class="row">
            <div class="col-sm-3">
              <ul>
                <li v-for="(item, index) in data.opcionesBot" :key="index">
                  <span>{{ index }}: {{ item }}</span>
                </li>
              </ul>
            </div>
            <div class="col-sm-5">
              <ul>
                <li v-for="(item, index) in data.posiciones" :key="index">
                  <span
                    >{{ get_name_of_symbol(index) }}:
                    <span v-for="(item2, index2) in item" :key="index2">
                      &nbsp;
                      <span>{{ index2 }}: {{ item2 }}</span>
                    </span>
                  </span>
                </li>
              </ul>
            </div>
          </div>
        </BaseBlock>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-8">
        <BaseBlock title="Book" btn-option-content ref="blockBookMasOrdenes">
          <template #options>
            <button type="button" class="btn-block-option">
              <i class="si si-refresh"></i>
            </button>
          </template>
          <div class="row">
            <div class="col-sm-4">
              <h6 class="border-bottom pb-2">Symbol</h6>
            </div>
            <div class="col-sm-2">
              <h6 class="border-bottom pb-2">Vol</h6>
            </div>
            <div class="col-sm-2">
              <h6 class="border-bottom pb-2">BI</h6>
            </div>
            <div class="col-sm-2">
              <h6 class="border-bottom pb-2">OF</h6>
            </div>
            <div class="col-sm-2">
              <h6 class="border-bottom pb-2">Vol</h6>
            </div>
          </div>
          <div
            class="row border-bottom pt-2"
            v-for="(item, index) in data.arrayBook"
            :key="index"
          >
            <div class="col-sm-4">
              <h6 class="pb-2">{{ get_name_of_symbol(index) }}</h6>
            </div>
            <div class="col-sm-2">
              <h6
                class="pb-2"
                v-for="(itemBI, index2) in item.BI"
                :key="index2"
              >
                {{ itemBI.size }}
              </h6>
            </div>
            <div class="col-sm-2">
              <h6
                class="pb-2"
                v-for="(itemBI, index3) in item.BI"
                :key="index3"
                v-html="verify_pendiente(index, itemBI.price, 1)"
              ></h6>
            </div>
            <div class="col-sm-2">
              <h6
                class="pb-2"
                v-for="(itemOF, index2) in item.OF"
                :key="index2"
                v-html="verify_pendiente(index, itemOF.price, 2)"
              ></h6>
            </div>
            <div class="col-sm-2">
              <h6
                class="pb-2"
                v-for="(itemOF, index3) in item.OF"
                :key="index3"
              >
                {{ itemOF.size }}
              </h6>
            </div>
          </div>
        </BaseBlock>
        <h2 class="content-heading" v-if="data.type_bot == 1">
          Ganancia estimada
        </h2>
        <div class="row" v-if="data.type_bot == 1">
          <div class="col-sm-6">
            <BaseBlock content-full ribbon="V 48 / C CI">
              <div class="text-center py-4">
                <p>Ganancia Estimada</p>
                <h4 class="mb-0">{{ get_ganancia_estimada(0) }}</h4>
              </div>
            </BaseBlock>
          </div>
          <div class="col-sm-6">
            <BaseBlock content-full ribbon="C 48 / V CI">
              <div class="text-center py-4">
                <p>Ganancia Estimada</p>
                <h4 class="mb-0">{{ get_ganancia_estimada(1) }}</h4>
              </div>
            </BaseBlock>
          </div>
        </div>
      </div>
      <div class="col-sm-4">
        <BaseBlock title="Ordenes" btn-option-content ref="blockOrdenes">
          <template #content>
            <ul class="nav nav-tabs nav-tabs-block" role="tablist">
              <li class="nav-item">
                <button
                  class="nav-link active"
                  id="btabs-static-home-tab"
                  data-bs-toggle="tab"
                  data-bs-target="#btabs-static-home"
                  role="tab"
                  aria-controls="btabs-static-home"
                  aria-selected="true"
                >
                  Pendientes
                </button>
              </li>
              <li class="nav-item">
                <button
                  class="nav-link"
                  id="btabs-static-profile-tab"
                  data-bs-toggle="tab"
                  data-bs-target="#btabs-static-profile"
                  role="tab"
                  aria-controls="btabs-static-profile"
                  aria-selected="false"
                >
                  Ejecutadas
                </button>
              </li>
              <li class="nav-item">
                <button
                  class="nav-link"
                  id="btabs-static-profile-tab"
                  data-bs-toggle="tab"
                  data-bs-target="#btabs-static-profile"
                  role="tab"
                  aria-controls="btabs-static-profile"
                  aria-selected="false"
                >
                  Canceladas
                </button>
              </li>
            </ul>
            <div class="block-content tab-content">
              <div
                class="tab-pane active pb-3"
                id="btabs-static-home"
                role="tabpanel"
                aria-labelledby="btabs-static-home-tab"
                tabindex="0"
              >
                <div
                  class="border p-2 mb-2"
                  v-for="(item, index) in data.arrayOrdenesPendientes"
                  :key="index"
                >
                  <h6 class="border-bottom pb-2">
                    {{ get_name_of_symbol(item.symbol) }}
                  </h6>
                  <div class="row">
                    <div class="col-sm-4">
                      <p>{{ item.date }}</p>
                    </div>
                    <div class="col-sm-8 pl-3">
                      <p style="color: green" v-if="item.side == 'Buy'">
                        Compra
                      </p>
                      <p style="color: red" v-else>Venta</p>
                      <p>{{ item.size }} x {{ item.price }}</p>
                    </div>
                  </div>
                </div>
              </div>
              <div
                class="tab-pane"
                id="btabs-static-profile"
                role="tabpanel"
                aria-labelledby="btabs-static-profile-tab"
                tabindex="0"
              >
                <div
                  class="border p-2 mb-2"
                  v-for="(item, index) in data.arrayOrdenesEjecutadas"
                  :key="index"
                >
                  <h6 class="border-bottom pb-2">{{ item.symbol }}</h6>
                  <div class="row">
                    <div class="col-sm-4">
                      <p>{{ item.date }}</p>
                    </div>
                    <div class="col-sm-8 pl-3">
                      <p style="color: green" v-if="item.side == 'Buy'">
                        Compra
                      </p>
                      <p style="color: red" v-else>Venta</p>
                      <p>{{ item.size }} x {{ item.price }}</p>
                    </div>
                  </div>
                </div>
              </div>
              <div
                class="tab-pane"
                id="btabs-static-settings"
                role="tabpanel"
                aria-labelledby="btabs-static-settings-tab"
                tabindex="0"
              >
                <h4 class="fw-normal">Settings Content</h4>
                <p>...</p>
              </div>
            </div>
          </template>
        </BaseBlock>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, onMounted, watch, computed, nextTick } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useTemplateStore } from "@/stores/template";
// Template components
import axios from "axios";
import Swal from "sweetalert2";
const route = useRoute();
const router = useRouter();
const id = computed(() => route.params.id);
const data = reactive({
  id_bot: 0,
  type_bot: null,
  symbols: [],
  arrayOrdenesPendientes: [],
  arrayOrdenesEjecutadas: [],
  arrayBook: [],
  opcionesBot: {},
  posiciones: {},
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

function get_name_of_symbol(symbol) {
  var textoSymbol = symbol.split("-");
  var texto = textoSymbol[2] + "-" + textoSymbol[3];
  return texto;
}

function findIndexArray(array, searchText, side) {
  return array.findIndex(function (item) {
    return item.symbol.includes(searchText) && item.side === side;
  });
}

function get_price_pendientes(symbol, side) {
  var price = 0;
  var index = findIndexArray(data.arrayOrdenesPendientes, symbol, side);
  if (index != -1) {
    price = data.arrayOrdenesPendientes[index].price;
  }
  return parseFloat(price);
}
function findObjectByKeyAndSide(object, keyText, side) {
  for (var key in object) {
    if (key.includes(keyText) && object[key][side].length > 0) {
      return object[key];
    }
  }
  return null;
}

function get_price_book(symbol, side) {
  var price = 0;
  var object = findObjectByKeyAndSide(data.arrayBook, symbol, side);
  if (object != null) {
    price = object[side][0].price;
  }
  return parseFloat(price);
}

function get_ganancia_estimada(type) {
  //necesito estas variables las voy a declarar para q no me genere el mensaje de error de q ya esta declarada -.-
  var V48 = 0;
  var CCI = 0;
  var C48 = 0;
  var VCI = 0;
  var ganancia = 0;
  if (type == 0) {
    //V 48 / C CI
    //necesito los datos de las ordenes pendientes de V48 y CCI
    V48 = get_price_pendientes("48", "Sell");
    CCI = get_price_pendientes("CI", "Buy");
    //necesito los datos del book contrarios C48 y VCI
    C48 = get_price_book("48", "BI");
    VCI = get_price_book("CI", "OF");
    //ahora hago una resta de los datos de las ordenes pendientes con los datos del book contrarios
    //y obtengo la ganancia estimada
    console.log("V48: " + parseFloat(V48));
    console.log("CCI: " + CCI);
    console.log("C48: " + C48);
    console.log("VCI: " + VCI);
    if (V48 > 0 && CCI > 0 && C48 > 0 && VCI > 0) {
      ganancia = V48 / CCI - C48 / VCI;
    }

    return ganancia;
  }
  if (type == 1) {
    //C48/VCI
    //necesito los datos de las ordenes pendientes de C48 y VCI
    C48 = get_price_pendientes("48", "Buy");
    VCI = get_price_pendientes("CI", "Sell");
    //necesito los datos del book contrarios C48 y VCI
    V48 = get_price_book("48", "OF");
    CCI = get_price_book("CI", "BI");
    //ahora hago una resta de los datos de las ordenes pendientes con los datos del book contrarios
    //y obtengo la ganancia estimada
    console.log("V48: " + parseFloat(V48));
    console.log("CCI: " + CCI);
    console.log("C48: " + C48);
    console.log("VCI: " + VCI);
    if (V48 > 0 && CCI > 0 && C48 > 0 && VCI > 0) {
      ganancia = C48 / VCI - V48 / CCI;
    }
    console.log("ganancia: " + ganancia);
    return ganancia;
  }
}

function verify_pendiente(symbol, price, side) {
  console.log("symbol", symbol);
  var result = "<span >" + price + "</span>";
  data.arrayOrdenesPendientes.forEach((item) => {
    console.log(parseFloat(item.price), price);
    if (parseFloat(item.price) == price && item.symbol == symbol) {
      if (side == 1) {
        result =
          "<span style='color:green'>" + price + "(" + item.size + ")</span>";
      } else {
        result =
          "<span style='color:red'>" + price + "(" + item.size + ")</span>";
      }

      return;
    }
  });
  console.log("result", result);
  return result;
}

async function get_bot_data(id) {
  var sesionFix = 0;
  if (localStorage.sesionFix) {
    sesionFix = localStorage.sesionFix;
  }
  await axios
    .post(store.urlBackend + "/bot_data_charts/" + id, {
      id_fix: sesionFix,
    })
    .then((response) => {
      console.log("bot data", response.data);
      data.id_bot = response.data.id;
      data.type_bot = response.data.type;
      data.symbols = response.data.symbols;
      data.arrayOrdenesPendientes = response.data.ordenesPendientes;
      data.arrayOrdenesEjecutadas = response.data.ordenesEjecutadas;
      data.arrayBook = response.data.book;
      data.opcionesBot = response.data.opciones;
      data.posiciones = response.data.posiciones;
      nextTick(() => {
        console.log("nextTick", data);
      });
    });
}

onMounted(async () => {
  console.log("router id  ", id.value);
  await get_bot_data(id.value);

  const socket = new WebSocket("ws://localhost:5100");

  socket.addEventListener("open", (event) => {
    socket.send("Hello Server!", event);
  });

  socket.addEventListener("message", (event) => {
    var dataBook = JSON.parse(event.data);
    console.log("Message from server ", dataBook);
    if (dataBook.type == "book") {
      for (var i = 0; i < data.symbols.length; i++) {
        if (dataBook.instrumentId.symbol == data.symbols[i]) {
          get_bot_data(id.value);
          break;
        }
      }
    }
  });
});
</script>
