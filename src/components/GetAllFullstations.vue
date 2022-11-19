<template>
    <v-responsive class="mx-auto" max-width="344">
        <v-btn @click="loadFuelstations()" variant="tonal"
          >Load fuelstations</v-btn
        >

        <div v-for="f in fuelstations">
           {{ f.id }} - {{ f.name }}
        </div>
    </v-responsive>
</template>
  
<script>
export default {
data: () => ({
    fuelstations: []
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
        .then((data) => this.fuelstations = data.data)
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
  