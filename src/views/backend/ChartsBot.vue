<template>
  <div>Hola</div>
</template>

<script setup>
import { reactive, onMounted, watch, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useTemplateStore } from "@/stores/template";
// Template components
import axios from "axios";
import Swal from "sweetalert2";
const route = useRoute();
const router = useRouter();
const id = computed(() => route.params.id);
const data = reactive({
  arrayDataBot: [],
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

async function get_bot_data(id) {
  await axios
    .get(store.urlBackend + "/bot_data_charts/" + id)
    .then((response) => {
      console.log("bot data", response);
      data.arrayDataBot = response.data;
    });
}

onMounted(async () => {
  console.log("router id  ", id.value);
  await get_bot_data(id.value);
});
</script>
