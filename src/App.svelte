<script>
  import { onMount, setContext } from "svelte";
  import { mapboxgl, keyApp } from "./mapbox.js";
  setContext(keyApp, {
    getMap: () => map,
  });
  export let map;
  let carDisplay = " ";
  let numDisplay = " ";
  let refDisplay = " ";
  let areaDisplay = " ";
  let zonaDisplay = " ";
  mapboxgl.accessToken =
    "pk.eyJ1IjoiYW5uYXBvcnRpIiwiYSI6ImNsMHVza2c2MzBuamkzY24xZHVjeDVyaTIifQ.3QTahE5rN_NBuT__v0VdGw";

  onMount(async () => {
    map = new mapboxgl.Map({
      container: "map",
      style: "https://geoserveis.icgc.cat/contextmaps/icgc_orto_estandard.json",
      center: [1.47, 41.27],
      zoom: 12.5,
    });

    map.addControl(new mapboxgl.NavigationControl());

    map.on("load", function () {
      map.setFog({
        range: [-1, 15.5],
        color: "white",
        "horizon-blend": 0.1,
      });

      addSources().then(function () {
        addSolarsLayer();
      });
    });

    map.on("styledata", function () {
      addSources().then(function () {
        addSolarsLayer();
      });
    });
  });

  async function addSources() {
    map.addSource("solars", {
      type: "geojson",
      data: "https://raw.githubusercontent.com/annaporti/solars/main/solars-bisbal.geojson",
      generateId: true,
    });
  }

  function addSolarsLayer() {
    if (!map.getLayer("solars")) {
      map.addLayer({
        id: "solars",
        type: "fill",
        source: "solars",
        layout: {
          visibility: "visible",
        },
        paint: {
          "fill-color": [
            "match",
            ["get", "ZONA"],
            "Can Gordei",
            "#fbb03b",
            "Nucli",
            "#00FFFF",
            "Papagai",
            "#e55e5e",
            "Miralba",
            "#3bb2d0",
            "Priorat",
            "#AAFF00",
            "Santa Cristina",
            "#228B22",
            "Esplai i Masieta",
            "#581845",
            "#FAFA33",
          ],
          "fill-opacity": [
            "case",
            ["boolean", ["feature-state", "hover"], false],
            1,
            0.5,
          ],
        },
        cursor: "pointer",
      });
      map.addLayer({
        id: "outline",
        type: "line",
        source: "solars",
        layout: {
          visibility: "visible",
        },
        paint: {
          "line-color": [
            "match",
            ["get", "ZONA"],
            "Can Gordei",
            "#fbb03b",
            "Nucli",
            "#00FFFF",
            "Papagai",
            "#e55e5e",
            "Miralba",
            "#3bb2d0",
            "Priorat",
            "#AAFF00",
            "Santa Cristina",
            "#228B22",
            "Esplai i Masieta",
            "#581845",
            "#FAFA33",
          ],
          "line-width": 3,
        },
      });
    }
    let hoveredStateId = null;

    map.on("mouseenter", "solars", (e) => {
      map.getCanvas().style.cursor = "pointer";
    });

    map.on("mousemove", "solars", (event) => {
      const maineCarrer = event.features[0].properties.CARRER;
      const maineNUM = event.features[0].properties.NUMERO;
      const maineRefcad = event.features[0].properties.REFCAT;
      const maineArea = event.features[0].properties.AREA;
      const maineZona = event.features[0].properties.ZONA;

      if (event.features.length === 0) return;

      carDisplay = maineCarrer;
      numDisplay = maineNUM;
      refDisplay = maineRefcad;
      areaDisplay = maineArea + " m²";
      zonaDisplay = maineZona;

      if (hoveredStateId) {
        map.removeFeatureState({
          source: "solars",
          id: hoveredStateId,
        });
      }

      hoveredStateId = event.features[0].id;

      map.setFeatureState(
        {
          source: "solars",
          id: hoveredStateId,
        },
        {
          hover: true,
        }
      );
    });

    map.on("mouseleave", "solars", () => {
      map.getCanvas().style.cursor = "";

      if (hoveredStateId) {
        map.setFeatureState(
          {
            source: "solars",
            id: hoveredStateId,
          },
          {
            hover: false,
          }
        );
      }

      hoveredStateId = null;

      carDisplay = " ";
      numDisplay = " ";
      refDisplay = " ";
      areaDisplay = " ";
      areaDisplay = " ";
    });
  }
</script>

<div id="map" />
<div class="info">
  <div><strong>Carrer:</strong> {carDisplay}</div>
  <div><strong>Número:</strong> {numDisplay}</div>
  <div><strong>Refrència cadastral:</strong> {refDisplay}</div>
  <div><strong>Àrea:</strong> {areaDisplay}</div>

</div>
<div class="legend">
  <p>
    <span class="legend-key" style="background-color:#fbb03b" /><span
      >Can Gordei</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#00FFFF" /><span
      >Nucli</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#e55e5e" /><span
      >Papagai</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#3bb2d0" /><span
      >Miralba</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#AAFF00" /><span
      >Priorat</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#228B22" /><span
      >Santa Cristina</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#581845" /><span
      >Esplai i Masieta</span
    >
  </p>
  <p>
    <span class="legend-key" style="background-color:#FAFA33" /><span
      >Ortigós</span
    >
  </p>
</div>

<style>
</style>
