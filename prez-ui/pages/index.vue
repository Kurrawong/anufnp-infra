<template>
  <NuxtLayout contentonly>
    <template #default>
      <div class="container mb-10">
        <h1 class="text-2xl mt-8">
          First Nations Portfolio Indigenous Research Portal
        </h1>
      </div>
      <div class="text-lg">
        <p class="mb-4">
          Use this portal to discover Indigenous research data held by the ANU.
        </p>
        <p class="mb-8">
          The catalogs hosted here expose historical research data from across
          the university.
        </p>
        <NuxtLink
          to="/catalogs"
          class="btn btn-primary px-4 py-2 rounded shadow bg-primary text-white hover:text-black hover:bg-secondary transition"
        >
          Browse the Portal &#8594;
        </NuxtLink>
      </div>
      <div class="text-lg mt-4 mb-4">
        <input
          id="search"
          class="px-4 py-2 rounded"
          type="text"
          placeholder="search place/language"
        />
      </div>
      <div id="map" class="h-96 mt-8 mx-auto pl-20 pr-20"></div>
    </template>
  </NuxtLayout>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import "ol/ol.css";
import Map from "ol/Map";
import View from "ol/View";
import TileLayer from "ol/layer/Tile";
import VectorLayer from "ol/layer/Vector.js";
import VectorSource from "ol/source/Vector.js";
import WKT from "ol/format/WKT.js";
import OSM from "ol/source/OSM";

const getTileSourceUrl = () => {
  const prefersDarkScheme = window.matchMedia(
    "(prefers-color-scheme: dark)",
  ).matches;
  return prefersDarkScheme
    ? "https://{a-c}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png"
    : "https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png";
};

const vectorSource = new VectorSource();
const vectorLayer = new VectorLayer({ source: vectorSource });

const loadFeatures = async (search_term) => {
  const query = `
  PREFIX geo: <http://www.opengis.net/ont/geosparql#>
  PREFIX schemas: <https://schema.org/>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
  select ?feature ?name ?wkt
  where {
    <https://anufncatalogue.anu.edu.au/refdata-geoms> rdfs:member ?feature .
    ?feature schemas:name ?name .
    ?feature geo:hasGeometry/geo:asWKT ?wkt .
    filter (contains(lcase(?name), "${search_term.toLowerCase()}"))
  }
  `;
  const url = new URL("http://localhost:8000/sparql");
  url.searchParams.append("query", query);
  const headers = new Headers();
  headers.append("content-type", "application/sparql-query");
  headers.append("accept", "application/sparql-results+json");
  try {
    const response = await fetch(url, { method: "GET", headers: headers });
    if (!response.ok) {
      console.log("Error retrieving data");
    }
    const data = await response.json();
    const results = data.results.bindings;
    vectorSource.clear();
    for (let i = 0; i < results.length; i++) {
      const result = results[i];
      const format = new WKT();
      const feature = format.readFeature(result.wkt.value, {
        dataProjection: "EPSG:4326",
        featureProjection: "EPSG:4326",
      });
      feature.setId(result.feature.value);
      feature.set("name", result.name.value);
      vectorSource.addFeature(feature);
    }
  } catch (error) {
    console.error("There was a problem with the fetch operation:", error);
  }
};

onMounted(() => {
  document.getElementById("search").addEventListener("change", (event) => {
    loadFeatures(event.target.value);
  });
  const map = new Map({
    target: "map",
    layers: [
      new TileLayer({
        source: new OSM({
          url: getTileSourceUrl(),
        }),
      }),
      vectorLayer,
    ],
    view: new View({
      center: [133, -25], // Centered on Australia
      zoom: 3.8,
      projection: "EPSG:4326",
    }),
  });
});
</script>
