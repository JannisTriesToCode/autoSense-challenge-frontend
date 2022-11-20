<template>
  <v-responsive class="mx-auto" max-width="600">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="id"
              label="ID"
              variant="underlined"
              :disabled="true"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="name"
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
              label="Address"
              variant="underlined"
              :disabled="true"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="city"
              label="City"
              variant="underlined"
              :disabled="true"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="latitude"
              label="Latitude"
              variant="underlined"
              :disabled="true"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="6">
            <v-text-field
              v-model="longitude"
              label="Longitude"
              variant="underlined"
              :disabled="true"
            ></v-text-field>
          </v-col>
        </v-row>

        <div class="pump" v-for="(pump, counter) in pumps" v-bind:key="counter">
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="pump.id"
                label="ID"
                variant="underlined"
                :disabled="true"
              ></v-text-field>
            </v-col>

            <v-col cols="12" md="6">
              <v-text-field
                v-model="pump.fuel_type"
                label="Fuel type"
                variant="underlined"
                :disabled="true"
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
                :disabled="true"
              ></v-checkbox>
            </v-col>
          </v-row>
        </div>

        <v-btn class="mr-4" @click="update(this.id)" variant="tonal">
          Update
        </v-btn>

        <v-btn class="mr-4" @click="loadFuelstation(this.id)" variant="tonal"
          >Load fuelstation</v-btn
        >

        <v-btn class="mr-4" @click="deleteFuelstation(this.id)" variant="tonal"
          >Delete fuelstation</v-btn
        >

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
      (v) => /^\d+(.\d+)?$/.test(v) || "Price must be a decimal number.",
    ],
  }),
  methods: {
    async update(id) {
      // This function updates a fuelstation to the backend
      const { valid } = await this.$refs.form.validate();

      if (valid) {
        await fetch(process.env.BACKEND_IP + "/fuelstations/" + id, {
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
          .then((response) => {
            if (response.ok) {
              return response.json();
            }
            throw new Error("Something went wrong");
          })
          .then(() => (this.error = null))
          .catch((error) => {
            this.error = error;
            console.log(error);
          });
      }
    },
    async loadFuelstation(id) {
      // This function gets the details of a fuelstation a loads it into the corresponding form
      await fetch(process.env.BACKEND_IP + "/fuelstations/" + id, {
        method: "GET",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => {
          if (response.ok) {
            return response.json();
          }
          throw new Error("Something went wrong");
        })
        .then((data) => {
          data = data.data;
          this.id = data.id;
          this.name = data.name;
          this.address = data.address;
          this.city = data.city;
          this.latitude = data.latitude.$numberDecimal;
          this.longitude = data.longitude.$numberDecimal;
          const temp_pumps = [];
          data.pumps.forEach((pump) => {
            temp_pumps.push({
              id: pump.id,
              fuel_type: pump.fuel_type,
              price: pump.price.$numberDecimal,
              available: pump.available
            })
          });
          this.pumps = temp_pumps;
          this.error = null;
        })
        .catch((error) => {
          this.error = error;
          console.log(error);
        });
    },
    async deleteFuelstation(id) {
      // This function deletes a fuelstation
      await fetch(process.env.BACKEND_IP + "/fuelstations/" + id, {
        method: "DELETE",
        headers: {
          "API-Key": process.env.API_KEY,
        },
      })
        .then((response) => {
          if (response.ok) {
            return response.json();
          }
          throw new Error("Something went wrong");
        })
        .then(() => {
          this.$refs.form.reset();
          this.marker.remove();
          this.error = null;
        })
        .catch((error) => {
          this.error = error;
          console.log(error);
        });
    },
  },
  mounted() {
    // Load the details of a fuelstation as soon as component is mounted
    this.loadFuelstation(this.id);
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
