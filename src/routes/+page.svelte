<script>

    import mapboxgl from "mapbox-gl";
    import * as d3 from 'd3';
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";

    mapboxgl.accessToken = "pk.eyJ1IjoiYnhpbWVuYWdvbWV6IiwiYSI6ImNtYXBodHd2dDBqNDQyanEzODlzcXZzdXUifQ.9K8alwSlgsUIZedt02AZ0w";

    import { onMount } from "svelte";

    let map;
    let stations = [];
    let mapViewChanged = 0;

    async function initMap() {
        map = new mapboxgl.Map({
            container: 'map',
            center: [-71.09415, 42.36027],
            zoom: 12,
            style: "mapbox://styles/mapbox/streets-v12",
        });
        await new Promise(resolve => map.on("load", resolve));
        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });

        map.addSource("cambridge_route", {
            type: "geojson",
            data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson",
        });

        map.addLayer({
            id: "SOME_ID", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                // paint params, e.g. colors, thickness, etc.
                "line-color": "green",
                "line-width": 3,
                "line-opacity": 0.4
            },

        });

        map.addLayer({
            id: "SOME_ID2", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "cambridge_route", // The id we specified in `addSource()`
            paint: {
                // paint params, e.g. colors, thickness, etc.
                "line-color": "green",
                "line-width": 3,
                "line-opacity": 0.4
            },

        });
    }

    async function loadStationData() {
    try {
        const csvUrl = 'https://vis-society.github.io/labs/8/data/bluebikes-stations.csv';
        const data = await d3.csv(csvUrl);
        
        stations = data.map(station => ({
            id: station.Number,
            name: station.NAME,
            Lat: +station.Lat,
            Long: +station.Long,
        }));
        // console.log('Stations loaded:', stations.length);
    } catch (error) {
        console.error('Error loading station data:', error);
    }
    }

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }

    

    onMount(() => {
        initMap();
        loadStationData();
    });
    
    $: map?.on("move", evt => mapViewChanged++);


</script>

<h1>Ximenas map</h1>
<p>completing o labotorios</p>

<div id="map">
	<svg>
        <g class="dots">
            {#key mapViewChanged}
                {#each stations as station}
                    <circle cx={ getCoords(station).cx }
                    cy={ getCoords(station).cy }
                    r="5" fill="steelblue" />

                {/each}

            {/key}
        </g>
        
    </svg>
</div>


<style>
    @import url("$lib/global.css");

</style>
