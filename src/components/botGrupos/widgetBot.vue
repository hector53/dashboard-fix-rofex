<template>
  <BaseBlock
    title=" Triangulo"
    :subtitle="ordenesBot.symbols"
    btn-option-content
    ref="blockBotOrdenes"
  >
    <template #options>
      <button
        type="button"
        class="btn-block-option"
        @click="updateData(ordenesBot.id)"
      >
        <i class="si si-refresh"></i>
      </button>
      <button
        type="button"
        class="btn-block-option"
        @click="eliminarBotDelGrupo(ordenesBot.id)"
      >
        <i class="fa fa-close"></i>
      </button>
      <button
        type="button"
        class="btn-block-option"
        @click="stopBot(ordenesBot.id)"
        v-if="data.statusBot == 1"
      >
        <i class="fa fa-stop"></i>
      </button>
      <button
        type="button"
        class="btn-block-option"
        @click="startBot(ordenesBot.id)"
        v-else
      >
        <i class="fa fa-play"></i>
      </button>
    </template>
    <table class="table table-vcenter">
      <thead>
        <tr>
          <th class="text-center">Symbol</th>
          <th class="text-center">Vol</th>
          <th class="text-center">BI</th>
          <th class="text-center">OF</th>
          <th class="text-center">Vol</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in data.ordenes" :key="index">
          <th class="text-center" scope="row">{{ item.symbol }}</th>
          <th class="text-center" scope="row">
            <span v-if="item.data[0] && item.data[0].side == 'Buy'">{{
              item.data[0].size
            }}</span>
            <span v-if="item.data[1] && item.data[1].side == 'Buy'">{{
              item.data[1].size
            }}</span>
          </th>
          <th class="text-center" scope="row">
            <span v-if="item.data[0] && item.data[0].side == 'Buy'">{{
              item.data[0].price
            }}</span>
            <span v-if="item.data[1] && item.data[1].side == 'Buy'">{{
              item.data[1].price
            }}</span>
          </th>
          <th class="text-center" scope="row">
            <span v-if="item.data[0] && item.data[0].side == 'Sell'">{{
              item.data[0].price
            }}</span>
            <span v-if="item.data[1] && item.data[1].side == 'Sell'">{{
              item.data[1].price
            }}</span>
          </th>
          <th class="text-center" scope="row">
            <span v-if="item.data[0] && item.data[0].side == 'Sell'">{{
              item.data[0].size
            }}</span>
            <span v-if="item.data[1] && item.data[1].side == 'Sell'">{{
              item.data[1].size
            }}</span>
          </th>
        </tr>
      </tbody>
    </table>
  </BaseBlock>
</template>

<script setup>
import { reactive, onMounted, ref, defineEmits } from "vue";
import axios from "axios";
import { useTemplateStore } from "@/stores/template";
const store = useTemplateStore();
const props = defineProps({
  ordenesBot: Object,
});
const emitters = defineEmits(["eliminar-bot-del-grupo"]);

const data = reactive({
  ordenes: [],
  statusBot: 0,
});
const blockBotOrdenes = ref(null);
console.log("componente widgetBot ", props.ordenesBot);

async function get_ordenes_by_props(arrayOrdenes) {
  let groupedArray = [];
  if (Array.isArray(arrayOrdenes)) {
    groupedArray = arrayOrdenes.reduce((acc, curr) => {
      const index = acc.findIndex((item) => item.symbol === curr.symbol);
      if (index === -1) {
        acc.push({ symbol: curr.symbol, data: [curr] });
      } else {
        acc[index].data.push(curr);
      }
      return acc;
    }, []);
  } else {
    console.error("props.objeto is not an array");
  }
  return groupedArray;
}

async function updateDataBot(item) {
  console.log("actualizando data bot", item);
  data.ordenes = await get_ordenes_by_props(item.ordenes);
  data.statusBot = item.status;
  console.log("ordenes", data.ordenes);
}
function startBot(id) {
  var id_fix = localStorage.sesionFix;
  store.pageLoader({ mode: "on" });
  axios
    .post(store.urlBackend + "/startBot", {
      id: id,
      id_fix: id_fix,
    })
    .then((response) => {
      console.log(response);
      updateData(id);
      store.pageLoader({ mode: "off" });
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
    });
}
async function stopBot(id) {
  var id_fix = localStorage.sesionFix;
  console.log("id_fix", id_fix);
  store.pageLoader({ mode: "on" });
  await axios
    .post(store.urlBackend + "/detenerBot", {
      id: id,
      id_fix: id_fix,
    })
    .then((response) => {
      console.log(response);
      updateData(id);
      store.pageLoader({ mode: "off" });
    })
    .catch((error) => {
      console.log(error);
      store.pageLoader({ mode: "off" });
    });
}

async function updateData(id) {
  console.log("actualizando data bot");
  // Set the block to loading state
  blockBotOrdenes.value.statusLoading();
  await axios
    .get(store.urlBackend + "/get_ordenes_bot_by_id/" + id)
    .then((response) => {
      updateDataBot(response.data[0]);
      blockBotOrdenes.value.statusNormal();
      console.log("bots ordenes", response);
    });
}

function eliminarBotDelGrupo(id_bot) {
  emitters("eliminar-bot-del-grupo", id_bot);
}

onMounted(async () => {
  data.ordenes = await get_ordenes_by_props(props.ordenesBot.ordenes);
  data.statusBot = props.ordenesBot.status;
  console.log("ordenes", data.ordenes);
});
</script>
