<template>
  <link
    href="https://unpkg.com/maplibre-gl@2.4.0/dist/maplibre-gl.css"
    rel="stylesheet"
  />
  <div id="map"></div>

  <!--
    <component class="variable-component" :is="changeableComponent" ></component>
    -->
</template>

<script>
import maplibregl from "maplibre-gl";
import ModifyFuelstationVue from "./ModifyFuelstation.vue";
import CreateFuelstationVue from "./CreateFuelstation.vue";

export default {
  data: () => ({
    map: null,
    fuelstations: null,
  }),
  methods: {
    async loadFuelstations() {
      await fetch(process.env.BACKEND_IP + "/fuelstations", {
        method: "GET",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          data.data.forEach((fuelstation) => {
            new maplibregl.Marker()
              .setLngLat([fuelstation.longitude, fuelstation.latitude])
              .setPopup(new maplibregl.Popup().setHTML(fuelstation.name))
              .addTo(this.map);
          });
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  components: {
    ModifyFuelstationVue,
    CreateFuelstationVue,
  },
  mounted() {
    this.map = new maplibregl.Map({
      container: "map",
      style:
        "https://api.maptiler.com/maps/streets/style.json?key=7rzbhFvyZxLBRQi4Hqw3",
      center: [8.52981, 47.3943939],
      zoom: 10,
    });

    this.map.on("load", () => {
      this.loadFuelstations();
    });
  },
  computed: {
    changeableComponent() {
      return "ModifyFuelstationVue";
    },
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}
.maplibregl-marker {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  border: 1px solid gray;
  background-color: yellow;
}
.variable-component {
  margin: 10%;
  right: 30%;
}
</style>
