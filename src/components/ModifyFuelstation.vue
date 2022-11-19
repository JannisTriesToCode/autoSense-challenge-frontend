<template>
  <v-responsive class="mx-auto" max-width="344">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-text-field
        v-model="id"
        label="ID"
        variant="underlined"
        :disabled="true"
      ></v-text-field>

      <v-text-field
        v-model="name"
        label="Name"
        variant="underlined"
        required
      ></v-text-field>

      <v-text-field
        v-model="address"
        label="Address"
        variant="underlined"
        :disabled="true"
      ></v-text-field>

      <v-text-field
        v-model="city"
        label="City"
        variant="underlined"
        :disabled="true"
      ></v-text-field>

      <v-text-field
        v-model="latitude"
        label="Latitude"
        variant="underlined"
        :disabled="true"
      ></v-text-field>

      <v-text-field
        v-model="longitude"
        label="Longitude"
        variant="underlined"
        :disabled="true"
      ></v-text-field>

      <div class="pump" v-for="(pump, counter) in pumps" v-bind:key="counter">
        <v-text-field
          v-model="pump.id"
          label="ID"
          variant="underlined"
          :disabled="true"
        ></v-text-field>

        <v-text-field
          v-model="pump.fuel_type"
          label="Fuel type"
          variant="underlined"
          :disabled="true"
        ></v-text-field>

        <v-text-field
          v-model="pump.price"
          :rules="priceRules"
          label="Price"
          variant="underlined"
          required
        ></v-text-field>

        <v-checkbox
          v-model="pump.available"
          label="Available"
          :disabled="true"
        ></v-checkbox>
      </div>

      <v-btn class="mr-4" @click="submit" variant="tonal"> Submit </v-btn>
      <v-btn @click="loadFuelstation('MIGROL_100042')" variant="tonal"
        >Load fuelstation</v-btn
      >
      <v-btn @click="deleteFuelstation('MIGROL_100042')" variant="tonal"
        >Delete fuelstation</v-btn
      >
    </v-form>
  </v-responsive>
</template>

<script>
export default {
  data: () => ({
    valid: true,
    id: null,
    name: null,
    nameRules: [(v) => !!v || "Name is required."],
    address: null,
    city: null,
    latitude: null,
    longitude: null,
    pumps: [{ id: null, fuel_type: null, price: null, available: null }],
    priceRules: [
      (v) => !!v || "ID is required.",
      (v) => /\d+.\d+/.test(v) || "Price must be a decimal number.",
    ],
  }),
  methods: {
    async submit() {
      const { valid } = await this.$refs.form.validate();

      if (valid) {
        await fetch(process.env.BACKEND_IP + "/fuelstations/" + this.id, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
            "API-Key": process.env.API_KEY,
          },
          body: JSON.stringify({
            name: this.name,
            pumps: this.pumps,
          }),
        })
          .then((response) => response.json())
          .then((data) => console.log(data))
          .catch((error) => {
            console.log(error);
          });
      }
    },
    async loadFuelstation(id) {
      await fetch(process.env.BACKEND_IP + "/fuelstations/" + id, {
        method: "GET",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          data = data.data;
          this.id = data.id;
          this.name = data.name;
          this.address = data.address;
          this.city = data.city;
          this.latitude = data.latitude;
          this.longitude = data.longitude;
          this.pumps = data.pumps;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    async deleteFuelstation(id) {
      await fetch(process.env.BACKEND_IP + "/fuelstations/" + id, {
        method: "DELETE",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => response.json())
        .then((data) => {
          console.log(data)
          this.$refs.form.reset()
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  
};
</script>

<style>
.pump {
  border: 1px solid;
  padding: 5px;
  margin-bottom: 10px;
}
</style>
