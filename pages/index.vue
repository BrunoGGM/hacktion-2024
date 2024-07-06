<template>
  <div class="grid grid-cols-12 gap-4 p-6">
    <div
      class="card bg-base-100 col-span-12 md:col-span-4 shadow-xl min-h-screen"
    >
      <div class="card-body p-0">
        <span class="text-2xl font-bold">Destinos</span>

        <draggable
          class="list-group"
          :list="places"
          group="places"
          @change="log"
          itemKey="id"
        >
          <template #item="{ element, index }">
            <div class="list-group-item">
              {{ element.properties.Place.title[0].text.content }}
            </div>
          </template>
        </draggable>
      </div>
    </div>
    <div
      class="card bg-base-100 col-span-12 md:col-span-4 shadow-xl min-h-screen"
    >
      <div class="card-body p-0">
        <span class="text-2xl font-bold"> Itinerario </span>
        <draggable
          class="list-group"
          :list="selectedPlaces"
          group="places"
          @change="log"
          itemKey="id"
          @add="addPlace"
        >
          <template #item="{ element, index }">
            <div class="list-group-item">
              {{ index + 1 }} -
              {{ element.properties.Place.title[0].text.content }}
            </div>
          </template>
        </draggable>
        <button
          v-if="selectedPlaces.length > 0"
          class="btn btn-primary"
          @click="viewRoute = true"
        >
          Calcular Costo
        </button>
      </div>
    </div>
    <div
      class="card bg-base-100 col-span-12 md:col-span-4 shadow-xl min-h-screen"
    >
      <div class="card-body py-2">
        <span class="text-2xl font-bold">Ruta</span>
        <div v-if="viewRoute">
          <ul class="steps steps-vertical">
            <li class="step step-primary">Inicio: Hotel</li>
            <li
              class="step step-primary"
              v-for="(place, index) in selectedPlaces"
              :key="index"
            >
              <div
                class="flex flex-col items-start justify-between w-full p-2 bg-base-200 rounded-lg"
              >
                <div>{{ place.properties.Place.title[0].text.content }}</div>
                <div class="text-xs">
                  Costo aproximado: ${{ Math.floor(Math.random() * 100) }}
                </div>
                <div class="text-xs">
                  Duración: {{ Math.floor(Math.random() * 10) }} minutos
                </div>
              </div>
            </li>
            <li class="step step-primary">Fin: Hotel</li>
          </ul>
          <div class="text-2xl font-bold text-center mt-4 rounded-lg p-2">
            Costo total: ${{ Math.floor(Math.random() * 100) }}
          </div>

          <iframe
            class="w-full mt-4 rounded-lg"
            src="https://www.google.com/maps/embed?pb=!1m14!1m12!1m3!1d25924.28262226077!2d139.74466958843215!3d35.68844201873905!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!5e0!3m2!1sen!2smx!4v1720307891810!5m2!1sen!2smx"
            height="450"
            style="border: 0"
            allowfullscreen=""
            loading="lazy"
            referrerpolicy="no-referrer-when-downgrade"
          ></iframe>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import draggable from "vuedraggable";
import { useNotionStore } from "~/stores/notion";
const notionStore = useNotionStore();
const places = computed(() => notionStore.places.results);
const selectedPlaces = ref([]);
const viewRoute = ref(false);

const log = (event) => {
  console.log(event);
};

const addPlace = (place) => {
  viewRoute.value = false;
};
function fetchPlaces() {
  $fetch(
    "https://api.notion.com/v1/databases/7c47628506f34218b7cf09e0f33715f4/query",
    {
      method: "POST",
      body: {},
      headers: {
        "Content-Type": "application/json",
        Authorization: "Bearer " + process.env.NOTION_API_KEY,
        "Notion-Version": "2022-06-28",
      },
    }
  )
    .then((response) => {
      places.value.response = response;
    })
    .catch((error) => {
      console.error(error);
    })
    .finally(() => {
      places.value.loading = false;
      places.value.loaded = true;
    });
}

onMounted(() => {
  //fetchPlaces();
});
</script>

<style>
.list-group {
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0;
  list-style-type: none;
  min-height: 200px;
}

.list-group-item {
  cursor: grab;
  min-height: 40px;
  border: 1px solid #2e363f;
  border-radius: 0.375rem;
  margin-bottom: 10px;
  padding: 8px 16px;
}
.list-group-item:active {
  cursor: grabbing;
}
</style>