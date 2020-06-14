<template>
  <div id="map-content">
    <div id="map"></div>
  </div>
</template>

<script>
import MarkerClusterer from '@google/markerclusterer'

const loadGoogleMapsApi = require('load-google-maps-api')

export default {
  name: 'HomePage',
  data () {
    return {
      googleMap: null,
      map: null,
      locations: [],
      userLocation: null,
      markerCluster: null,
      markers: null,
      userMarker: null,
      userCircleMarker: null
    }
  },
  async mounted () {
    this.locations = this.$store.state.marker.markers.slice()

    await this.createMap()
    await this.updateUserMarker()

    //現在地を追跡する
    navigator.geolocation.watchPosition(pos => {
      this.userLocation = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude
      }
      //元の文
    //navigator.geolocation.getCurrentPosition(pos => {
      //this.userLocation = {
        //lat: pos.coords.latitude,
        //lng: pos.coords.longitude
      //}

      this.updateUserMarker()
    })
  },
  methods: {
    /**
     * GoogleMap 作成
     * @returns {Promise<void>}
     */
    async createMap () {
      this.googleMap = await loadGoogleMapsApi({ key: process.env.GOOGLE_API_KEY })
      this.map = new this.googleMap.Map(document.getElementById('map'), {
        center: new this.googleMap.LatLng(35.6130473, 140.1067103),
        zoom: 13
      })

      await this.updateMarker()
    },
    /**
     * マップに対してマーカー追加
     * @returns {Promise<void>}
     */
    async updateMarker () {
      if (!this.googleMap) return
      if (this.markers && this.markerCluster) {
        this.markerCluster.removeMarkers(this.markers)
      }

      const infoWindow = new this.googleMap.InfoWindow()

      const googleMap = this.googleMap
      const map = this.map

      this.markers = this.locations.map(function (loc) {
        const marker = new googleMap.Marker({
          position: {
            lat: parseFloat(loc.lat),
            lng: parseFloat(loc.lng),
            title: loc.title
          },
          zIndex: 1
        })

        googleMap.event.addListener(marker, 'click', () => {
          infoWindow.setContent(loc.title)
          infoWindow.open(map, marker)
        })

        return marker
      })

      this.markerCluster = new MarkerClusterer(map, this.markers, {
        imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m'
      })
    },
    async updateUserMarker () {
      if (!this.userLocation) return

      const latLng = new this.googleMap.LatLng(this.userLocation.lat, this.userLocation.lng)

      this.userMarker = this.userMarker || new this.googleMap.Marker({
        position: latLng,
        zIndex: 1
      })

      // 既にマップに表示していたら非表示
      this.userMarker.setMap(null)

      this.userMarker.setPosition(latLng)

      if (this.map) {
        this.userMarker.setMap(this.map)
        this.map.panTo(latLng)
      }

      const infoWindow = new this.googleMap.InfoWindow()

      this.googleMap.event.addListener(this.userMarker, 'click', () => {
        infoWindow.setContent('現在位置')
        infoWindow.open(this.map, this.userMarker)
      })
    },
    async updateUserCircleMarker () {
      const latLng = new this.googleMap.LatLng(this.userLocation.lat, this.userLocation.lng)

      this.userCircleMarker = this.userCircleMarker || new this.googleMap.Circle({
        center: latLng,
        radius: 50,
        strokeColor: '#0088ff',
        strokeOpacity: 0.8,
        strokeWeight: 1,
        fillColor: '#0088ff',
        fillOpacity: 0.2
      })

      this.userCircleMarker.setMap(this.map)
    }
  },
  computed: {
    dataList () {
      return this.$store.state.marker.markers.slice()
    }
  },
  watch: {
    dataList (val, old) {
      this.locations = val
      this.updateMarker()
    }
  }
}
</script>

<style>
#map-content, #map {
  height: 100%;
}
</style>
