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
        <div id="map" class="h-96 mt-8 mx-auto pl-20 pr-20"></div>
      </div>
    </template>
  </NuxtLayout>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import "ol/ol.css";
import Map from "ol/Map";
import View from "ol/View";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";

const getTileSourceUrl = () => {
  const prefersDarkScheme = window.matchMedia(
    "(prefers-color-scheme: dark)",
  ).matches;
  return prefersDarkScheme
    ? "https://{a-c}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png"
    : "https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png";
};

onMounted(() => {
  const map = new Map({
    target: "map",
    layers: [
      new TileLayer({
        source: new OSM({
          url: getTileSourceUrl(),
        }),
      }),
    ],
    view: new View({
      center: [133, -25], // Centered on Australia
      zoom: 3.8,
      projection: "EPSG:4326",
    }),
  });
});
</script>
