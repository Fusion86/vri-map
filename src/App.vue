<template>
  <div id="app">
    <VriMap />
  </div>
</template>

<script>
import Vue from "vue";
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
import "leaflet/dist/leaflet.css";
import VriMap from "./components/VriMap.vue";

Vue.component("l-map", LMap);
Vue.component("l-tile-layer", LTileLayer);
Vue.component("l-marker", LMarker);

export default {
  name: "App",
  components: {
    VriMap,
  },
};
</script>

<style>
body {
  margin: 0;
}
</style>
