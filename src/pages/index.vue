<script setup lang="ts">
import { reactive } from 'vue'
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
import MapboxLanguage from '@mapbox/mapbox-gl-language'
import MapboxGeocoder from '@mapbox/mapbox-gl-geocoder'
import '@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css'
import geojson from '~/data/geojson.json'

mapboxgl.accessToken = ''

const lngLat = reactive({
  lng: 0,
  lat: 0,
})
function initMap() {
  const map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/satellite-streets-v11',
    center: [103.99414066575957, 30.482608517879285],
    zoom: 6,
    projection: 'globe',
  })
  if (mapboxgl.getRTLTextPluginStatus() !== 'loaded')
    mapboxgl.setRTLTextPlugin('https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-rtl-text/v0.2.3/mapbox-gl-rtl-text.js')
  map.addControl(new MapboxLanguage({
    defaultLanguage: 'zh-Hans',
  }))

  map.on('click', (e: any) => {
    const { lng, lat } = e.lngLat
    lngLat.lng = lng
    lngLat.lat = lat
    // popup = new mapboxgl.Popup({ closeOnClick: false })
    //   .setLngLat([lng, lat])
    //   .setHTML(`<h1 style="color: #121212">${lng} ${lat}</h1>`)
    //   .addTo(map)
  })
  for (const marker of geojson.features) {
    const el = document.createElement('img')
    const width = marker.properties.iconSize[0]
    const height = marker.properties.iconSize[1]
    el.className = 'marker'
    el.src = getImageUrl(marker.properties.name)
    el.style.width = `${width}px`
    el.style.height = `${height}px`
    el.style.backgroundSize = '100%'
    // Add markers to the map.
    new mapboxgl.Marker(el)
      .setLngLat(marker.geometry.coordinates)
      .setPopup(new mapboxgl.Popup({ offset: 25 }).setHTML(`<h3 class="marker-popup-title">${marker.title}</h3>`))
      .addTo(map)
  }
  // lng: 106.47495136721426, lat: 29.718950557366156

  // map.addControl(new MapboxGeocoder({
  //   accessToken: mapboxgl.accessToken,
  //   mapboxgl: mapboxgl,
  // }))

  // map.addControl(
  //   new mapboxgl.GeolocateControl({
  //     positionOptions: {
  //       enableHighAccuracy: true,
  //     },
  //     trackUserLocation: true,
  //     showUserHeading: true,
  //   }))
}

function getImageUrl(name: string) {
  return new URL(`../assets/markerImgs/${name}.svg`, import.meta.url).href
}

onMounted(() => {
  initMap()
})
</script>

<template>
  <div id="map" style="height: 100vh;" />
  <div class="map-info">
    <ul>
      <li>经纬度: {{ lngLat.lng }}, {{ lngLat.lat }}</li>
    </ul>
  </div>
</template>

<style>
.mapboxgl-ctrl-bottom-left {
  display: none;
}

.mapboxgl-ctrl-bottom-right {
  display: none;
}

.mapboxgl-ctrl-top-right .mapboxgl-ctrl {
  margin-top: 60px;
}

.marker-popup-title {
  font-size: 12px;
  font-weight: 600;
  color: #121212;
}

.mapboxgl-popup-close-button {
  color: #000;
}

.map-info {
  position: absolute;
  top: 0;
  font-family: sans-serif;
  margin-top: 5px;
  margin-left: 5px;
  padding: 5px;
  font-size: 12px;
  color: #222;
  background-color: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(4px);
  border-radius: 5px;
}
</style>
