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
          Browse the Catalogs &#8594;
        </NuxtLink>
      </div>
      <div class="text-lg mt-4 mb-4">
        <input
          id="search"
          class="px-4 py-1 rounded mr-8 border"
          type="text"
          placeholder="Search by place/language"
        />
        <span id="spinner" class="spinner"></span>
      </div>
      <div class="flex mt-8">
        <div id="resultlist" class="w-1/4 h-96 overflow-y-auto pr-4">
          <ul>
            <li class="mt-4 text-sm text-gray-500 italic">
              Use the map to search for and select a language group or place
              name to see resources that mention it
            </li>
          </ul>
        </div>
        <div id="map" class="h-96 w-3/4"></div>
      </div>
    </template>
  </NuxtLayout>
</template>

<script setup>
import { onMounted } from "vue";
import "ol/ol.css";
import Map from "ol/Map";
import View from "ol/View";
import TileLayer from "ol/layer/Tile";
import VectorLayer from "ol/layer/Vector.js";
import VectorSource from "ol/source/Vector.js";
import WKT from "ol/format/WKT.js";
import OSM from "ol/source/OSM";

const runtimeConfig = useRuntimeConfig();

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
  if (search_term.length < 1) {
    return;
  }
  spinner.style.display = "inline-block";
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
  const url = new URL(`${runtimeConfig.public.prezApiEndpoint}/sparql`);
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
    spinner.style.display = "none";
  } catch (error) {
    spinner.style.display = "none";
    console.error("There was a problem with the fetch operation:", error);
  }
};

onMounted(() => {
  const spinner = document.getElementById("spinner");
  spinner.style.display = "none";
  const resultList = document.getElementById("resultlist");
  const handleFeatureClick = async (featureId, featureName) => {
    let ul = document.createElement("ul");
    let liheader = document.createElement("li");
    liheader.innerHTML = `<li class="mb-4 mt-4">Resources mentioning <a href="${featureId}" target="_blank">${featureName}</a></li>`;
    ul.appendChild(liheader);
    const query = `
    PREFIX schema: <https://schema.org/>
    select ?thing (coalesce(?label) as ?name)
    where {
      ?thing schema:mentions <${featureId}> .
      values ?label_prop {
        schema:name schema:title schema:headline 
      }
      ?thing ?label_prop ?label .
    }`;
    const url = new URL(`${runtimeConfig.public.prezApiEndpoint}/sparql`);
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
      if (results.length < 1) {
        let li = document.createElement("li");
        li.innerHTML = `<li class="mt-4 text-sm text-gray-500 italic">No results for ${featureName}</li>`;
        ul.appendChild(li);
        resultList.appendChild(ul);
      }
      for (let i = 0; i < results.length; i++) {
        const result = results[i];
        let li = document.createElement("li");
        li.innerHTML = `&#8594; <a href="${window.location}object?uri=${result.thing.value}" target="_blank">${result.name.value}</a>`;
        ul.appendChild(li);
        resultList.appendChild(ul);
      }
    } catch (error) {
      console.error("There was a problem with the fetch operation:", error);
    }
  };
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
  // Create a tooltip element
  const tooltipElement = document.createElement("div");
  tooltipElement.className =
    "maptt px-2 py-1 border rounded bg-primary text-white";
  tooltipElement.style.display = "none";
  tooltipElement.style.position = "absolute";
  tooltipElement.style.pointerEvents = "none";
  document.body.appendChild(tooltipElement);

  // Add event listener for pointer move
  map.on("pointermove", function (evt) {
    const feature = map.forEachFeatureAtPixel(evt.pixel, function (feature) {
      return feature;
    });

    if (feature) {
      tooltipElement.innerHTML = feature.get("name");
      tooltipElement.style.display = "block";
      const mapRect = map.getTargetElement().getBoundingClientRect();
      tooltipElement.style.left =
        mapRect.left + evt.pixel[0] + window.scrollX - 30 + "px";
      tooltipElement.style.top =
        mapRect.top + evt.pixel[1] + window.scrollY - 50 + "px";
    } else {
      tooltipElement.style.display = "none";
    }
  });
  // Add click event listener to the map
  map.on("singleclick", function (event) {
    resultList.innerHTML = "";
    map.forEachFeatureAtPixel(event.pixel, function (feature) {
      const featureId = feature.getId();
      const featureName = feature.get("name");
      if (featureId) {
        handleFeatureClick(featureId, featureName);
      }
    });
  });
});
</script>
<style scoped>
.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border-left-color: var(--bg-secondary);
  animation: spin 1s ease infinite;
  margin: 0 auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
