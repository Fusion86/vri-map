<template>
  <l-map
    id="map"
    ref="map"
    :zoom="13"
    :center="center"
    :options="{preferCanvas: true}"
    @ready="onMapReady()"
    @update:zoom="onMapZoom"
  >
    <l-tile-layer :url="url" :attribution="attribution" />
  </l-map>
</template>

<script>
import L from "leaflet";
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
import eol from "eol";
import randomcolor from "randomcolor";

class VriSystem {
  constructor(str) {
    const parts = str.split(";");

    if (parts.length != 24) {
      throw "parts.length != 24";
    }

    this.region = parts[1];
    this.id = parts[2];

    if (this.id.length == 0) {
      throw "Empty ID";
    }

    if (parts[5].length > 0 && parts[6].length > 0) {
      this.location = L.latLng(
        parseFloat(parts[5].replace(",", ".")),
        parseFloat(parts[6].replace(",", "."))
      );
    }

    this.type = parts[7];
  }
}

export default {
  name: "VriMap",
  components: {
    LMap,
    LTileLayer,
    LMarker,
  },
  data() {
    return {
      map: null,
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 13,
      center: L.latLng(51.7, 5.3),
      vri_systems: [],
      region_color_map: {},
      markers: [],
    };
  },
  methods: {
    onMapReady() {
      this.map = this.$refs.map.mapObject;
    },
    async loadData() {
      // Fetch data from file on server.
      const resp = await fetch("./data/TLC-ID.csv");

      if (!resp.ok) {
        throw "Couldn't fetch data!";
      }

      const text = await resp.text();

      const lines = eol.split(text); // Split on newlines.
      lines.shift(); // Remove first line.

      // Clear old data
      this.vri_systems = [];

      for (const line of lines) {
        try {
          const vri = new VriSystem(line);
          this.vri_systems.push(vri);
        } catch (ex) {
          console.warn("Not adding VRI System. Reason: " + ex);
        }
      }

      // Generate colors for regions.
      const regions = [...new Set(this.vri_systems.map((x) => x.region))];

      // Clear old colormap and generate new colors.
      this.region_color_map = {};
      for (const region of regions) {
        this.region_color_map[region] = randomcolor({
          luminosity: "dark",
          seed: region,
        });
      }
    },
    drawMarkers() {
      console.log("drawMarkers start");
      const start = new Date();

      // Remove old markers.
      let old_marker;
      while ((old_marker = this.markers.pop())) {
        this.map.removeLayer(old_marker);
      }

      // Add new markers.
      for (const vri of this.vri_systems) {
        if (vri.location != undefined) {
          let radius = 2;
          if (this.zoom < 15) {
            radius = 40;
          } else if (this.zoom < 18) {
            radius = 10;
          }

          const marker = L.circle(vri.location, {
            color: this.region_color_map[vri.region],
            radius: radius,
          }).addTo(this.map);
          this.markers.push(marker);

          let html = `ID: ${vri.id}<br>Regio: ${vri.region}`;
          marker.bindPopup(html);
        }
      }

      // Execution time.
      const end = new Date() - start;
      console.info("Execution time: %dms", end);
    },
    onMapZoom(zoom) {
      console.log("zoom = " + zoom);

      if (this.zoom != zoom) {
        this.zoom = zoom;
        this.drawMarkers();
      }
    },
  },
  async mounted() {
    await this.loadData();
    this.drawMarkers();
  },
};
</script>

<style scoped>
#map {
  height: 100vh;
}
</style>
