<template>
  <link
    href="https://unpkg.com/maplibre-gl@2.4.0/dist/maplibre-gl.css"
    rel="stylesheet"
  />
  <div id="map"></div>

  <v-btn @click="toggleFuelstationCreate" variant="tonal"
    >Toggle Fuelstation Creation Mode</v-btn
  >
</template>

<script>
import maplibregl from "maplibre-gl";
import ModifyFuelstationVue from "./ModifyFuelstation.vue";
import CreateFuelstationVue from "./CreateFuelstation.vue";
import { createApp, defineComponent, nextTick } from "vue";
import { registerPlugins } from "@/plugins";

export default {
  data: () => ({
    creationMarker: null,
    creationMode: false,
    map: null,
    fuelstations: null,
  }),
  methods: {
    async loadFuelstations() {
      // This functions fetches all fuelstations and adds them to the map
      await fetch(process.env.BACKEND_IP + "/fuelstations", {
        method: "GET",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          // Add marker for each fuelstation
          data.data.forEach((fuelstation) => {
            const marker = new maplibregl.Marker()
              .setLngLat([fuelstation.longitude.$numberDecimal, fuelstation.latitude.$numberDecimal])
              .setPopup(
                new maplibregl.Popup().setHTML(
                  '<div id="popup-content-' + fuelstation.id + '"></div>'
                )
              )
              .addTo(this.map);

            // Marker popup html code only gets added to the DOM after it has been toggled once
            marker.togglePopup();
            // The popup of the fuelstation markers should contain the ModifyFuelstation component so
            // that we can see and modify their details
            const MyNewPopup = defineComponent({
              extends: ModifyFuelstationVue,
              setup() {
                const id = fuelstation.id;
                return { id, marker };
              },
            });
            nextTick(() => {
              const app = createApp(MyNewPopup);
              registerPlugins(app);
              app.mount("#popup-content-" + fuelstation.id);
            });
          });
        })
        .catch((error) => {
          console.log(error);
        });

      // click at random position to untoggle all popus
      document.elementFromPoint(0, 0).click();
    },
    toggleFuelstationCreate() {
      // This enables users to click anywhere on the map and create a new fuelstation
      this.creationMode = true;
    },
  },
  mounted() {
    // Define map
    this.map = new maplibregl.Map({
      container: "map",
      style:
        "https://api.maptiler.com/maps/streets/style.json?key=" + process.env.MAP_TILER_API_KEY,
      center: [8.52981, 47.3943939],
      zoom: 10,
    });

    // After map is loaded load fuelstations
    this.map.on("load", () => {
      this.loadFuelstations();
    });

    // If creationMode is active users can click on the map and create a new fuelstation
    this.map.on("click", (e) => {
      if (this.creationMode) {
        // Get coords of clicked location
        const latitude = e.lngLat.lat;
        const longitude = e.lngLat.lng;

        // Create marker at location
        const marker = new maplibregl.Marker({ color: "#AAAAAA" })
          .setLngLat([longitude, latitude])
          .setPopup(
            new maplibregl.Popup().setHTML('<div id="new-fuelstation"></div>')
          )
          .addTo(this.map);

        // Open popup and add the CreateFuelstation component to it
        marker.togglePopup();
        const MyNewPopup = defineComponent({
          extends: CreateFuelstationVue,
          setup() {
            return { marker, longitude, latitude };
          },
        });
        nextTick(() => {
          const app = createApp(MyNewPopup);
          registerPlugins(app);
          app.mount("#new-fuelstation");
        });

        // Save the creationMarker so that we can remove it later
        this.creationMarker = marker;
        // Set creationMode to false so that the next click doesnt create a new marker + fuelstation
        this.creationMode = false;
      } else {
        if (this.creationMarker) {
          // Remove creationMarker
          this.creationMarker.remove();
        }
      }
    });
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
.maplibregl-popup {
  max-width: 600px !important;
  width: 600px;
}
</style>
