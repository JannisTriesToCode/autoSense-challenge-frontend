<template>
  <v-responsive class="mx-auto" max-width="600">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="fuelstation_id"
              :rules="fuelstation_idRules"
              label="ID"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="name"
              :rules="nameRules"
              label="Name"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="address"
              :rules="addressRules"
              label="Address"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="city"
              :rules="cityRules"
              label="City"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="latitude"
              :rules="latitudeRules"
              label="Latitude"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="longitude"
              :rules="longitudeRules"
              label="Longitude"
              variant="underlined"
              required
            ></v-text-field>
          </v-col>
        </v-row>
        <div class="pump" v-for="(pump, counter) in pumps" v-bind:key="counter">
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="pump.id"
                :rules="pump_idRules"
                label="ID"
                variant="underlined"
                required
              ></v-text-field>
            </v-col>

            <v-col cols="12" md="6">
              <v-text-field
                v-model="pump.fuel_type"
                :rules="fuel_typeRules"
                label="Fuel type"
                variant="underlined"
                required
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="pump.price"
                :rules="priceRules"
                label="Price"
                variant="underlined"
                required
              ></v-text-field>
            </v-col>

            <v-col cols="12" md="6">
              <v-checkbox
                v-model="pump.available"
                label="Available"
              ></v-checkbox>
            </v-col>
          </v-row>
          <v-btn class="mr-4" @click="addPump" variant="tonal">
            New Pump
          </v-btn>

          <v-btn
            class="mr-4"
            @click="deletePump(counter)"
            color="error"
            variant="tonal"
          >
            Delete Pump
          </v-btn>
        </div>

        <v-btn class="mr-4" @click="submit" variant="tonal">
          Create Fuelstation
        </v-btn>

        <v-alert density="compact" type="error" icon="$info" v-show="error"
          >>{{ this.error }}</v-alert
        >
      </v-container>
    </v-form>
  </v-responsive>
</template>

<script>
export default {
  data: () => ({
    error: null,
    marker: null,
    valid: true,
    fuelstation_id: null,
    fuelstation_idRules: [(v) => !!v || "ID is required."],
    name: null,
    nameRules: [(v) => !!v || "Name is required."],
    address: null,
    addressRules: [(v) => !!v || "Address is required."],
    city: null,
    cityRules: [(v) => !!v || "City is required."],
    latitude: null,
    latitudeRules: [
      (v) => !!v || "Latitude is required.",
      (v) => /^\d+.\d+$/.test(v) || "Latitude must be a decimal number.",
    ],
    longitude: null,
    longitudeRules: [
      (v) => !!v || "Longitude is required.",
      (v) => /^\d+.\d+$/.test(v) || "Longitude must be a decimal number.",
    ],
    pumps: [{ id: null, fuel_type: null, price: null, available: false }],
    pump_idRules: [(v) => !!v || "ID is required."],
    fuel_typeRules: [(v) => !!v || "ID is required."],
    priceRules: [
      (v) => !!v || "ID is required.",
      (v) => /^\d+.\d+$/.test(v) || "Price must be a decimal number.",
    ],
  }),
  methods: {
    async submit() {
      const { valid } = await this.$refs.form.validate();

      if (valid) {
        await fetch(process.env.BACKEND_IP + "/fuelstations", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "API-Key": process.env.API_KEY,
          },
          body: JSON.stringify({
            id: this.fuelstation_id,
            name: this.name,
            address: this.address,
            city: this.city,
            latitude: this.latitude,
            longitude: this.longitude,
            pumps: this.pumps,
          }),
        })
          .then((response) => response.json())
          .then((responseJson) => {
            // Temp hack
            // Check if response has a message property. This indicates that the fuelstation id already exists.
            // TODO: Use response.status for this
            if (Object.prototype.hasOwnProperty.call(responseJson, "message")) {
              throw new Error(responseJson.message);
            } else {
              this.marker.remove();
              location.reload();
            }
          })
          .catch((error) => {
            this.error = error;
            console.log(error);
          });
      }
    },
    addPump: function () {
      this.pumps.push({
        pump_id: null,
        fuel_type: null,
        price: null,
        available: null,
      });
    },
    deletePump(counter) {
      if (counter > 0) {
        this.pumps.splice(counter, 1);
      }
    },
  },
};
</script>

<style>
.pump {
  border: 1px solid darkgray;
  padding: 5px;
  margin-bottom: 10px;
}
</style>
