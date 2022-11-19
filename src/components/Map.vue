<template>    
    <link href="https://unpkg.com/maplibre-gl@2.4.0/dist/maplibre-gl.css" rel="stylesheet" />
    <div id="map"></div>
</template>
  
<script>
import maplibregl from 'maplibre-gl'



export default {    
    data: () => ({        
    }),
    methods: {
    },
    mounted () {
        var map = new maplibregl.Map({
            container: 'map',
            style:'https://api.maptiler.com/maps/streets/style.json?key=7rzbhFvyZxLBRQi4Hqw3',
            center: [8.52981, 47.3943939],
            zoom: 10
        });

        map.on('load', () => {
            var marker = new maplibregl.Marker()
                .setLngLat([8.52981, 47.3943939])
                .addTo(map);            
        });

    } 
};
</script>

<style>
	body { margin: 0; padding: 0; }
	#map { position: absolute; top: 0; bottom: 0; width: 100%; }
    .maplibregl-marker {
        width: 25px;
        height: 25px;
        border-radius: 50%;
        border:1px solid gray;
        background-color:yellow;
    }
</style>
  