<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";

    import { onMount } from "svelte";

    import * as d3 from "d3";

    mapboxgl.accessToken = "pk.eyJ1IjoicGVkcm90b2thciIsImEiOiJjbWFwaDRrd28wZ3J6Mm1vOGxmdXdsNWE4In0.E-O7ttzdDJ-1YpdZx7RQ_Q";

    let map;
    let stations = [];
    let mapViewChanges = 0;

    $: map?.on("move", evt => mapViewChanges++)

    async function initMap() {
        map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/pedrotokar/cmaphfbgf01a901s87glm631n",
            zoom: 12,
            center: [-71.0788727679991, 42.36199480654666]
        });

        await new Promise(resolve => map.on("load", resolve));

        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });

        map.addLayer({
            id: "boston-bike-paths",
            type: "line",
            source: "boston_route",
            paint: {
                "line-color": "blue",
                "line-width": 3,
                "line-opacity": 0.5
            },
        });

        map.addSource("cambridge_route", {
            type: "geojson",
            data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson",
        });

        map.addLayer({
            id: "cambridge-bike-paths",
            type: "line",
            source: "cambridge_route",
            paint: {
                "line-color": "purple",
                "line-width": 3,
                "line-opacity": 0.5
            },
        });
    };

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
            console.log("Estações carregadas:", stations);
        } catch (error) {
            console.error("Erro ao carregar as estações:", error);
        }
    }

    onMount(() => {
        initMap();
        loadStationData();
    });

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }


</script>

<h1>Mapa de bicicletas</h1>
<p>Esse site fornece uma visualização em mapa do uso de bicicletas em Boston.</p>

<div id="map">
    <svg>
    {#key mapViewChanges}
        {#each stations as station}
            <circle {...getCoords(station)}
                    r="4"
                    fill="#eff319be" />
        {/each}
    {/key}
    </svg>
</div>

<style>
    @import url("$lib/global.css");

    #map {
        flex: 1;
    }

    #map svg {
        position: absolute;
        z-index: 1;

        width: 100%;
        height: 100%;

        pointer-events: none;
    }
</style>
