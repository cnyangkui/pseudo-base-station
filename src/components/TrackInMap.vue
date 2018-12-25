<template>
   <div id="mapid"></div>
</template>

<script>
import L from 'leaflet';
export default {
   props: ['defaultData'],
   data() {
      return {
         map: null,
         groupLayer: null,
         mapConfig: {
            zoom: 10,
            center: [39.92, 116.46],
            minZoom: 3,
            maxZoom: 18,
         },
      }
    },
   mounted() {
      this.$nextTick(() => {
         this.initMap();
         this.addMapLayer();
         this.addGroupLayer();
         this.clearMarkers();
         console.log('init........')
         if(this.defaultData != null) {
            this.addMarkers(this.defaultData);
            // this.addTrack(this.defaultData);
         }
      })
   },
   watch: {
      defaultData: function(newV, oldV) {
         this.clearMarkers();
         this.addMarkers(newV);
         // this.addTrack(newV);
      }
   },
   methods: {
       initMap() {
         this.map = L.map("mapid", {
            center: this.mapConfig.center,
            zoom: this.mapConfig.zoom,
            minZoom: this.mapConfig.minZoom,
            maxZoom: this.mapConfig.maxZoom,
            // attribution: "&copy; 高德地图"
         })
      },
      addMapLayer() {
         // L.titleLayer.mapProvider("GaoDe.Normal.Map", {
         //    attribution: this.mapConfig.attribution
         // }.addTo(this.map));
         this.tileLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
         }).addTo(this.map);
      },
      addGroupLayer() {
         this.groupLayer = new L.LayerGroup();
         this.map.addLayer(this.groupLayer);
      },
      addMarkers(dataset) {
         dataset.forEach(d => {
            let marker = L.circle([d.lat, d.lng], {
               color: 'blue',
               opacity: 0.05,
               radius: 50
            });
            this.groupLayer.addLayer(marker);  
         })
      },
      clearMarkers() {
         this.groupLayer.clearLayers();
      }
      // addTrack(dataset) {
      //    // let latlngs = filterData.map(d => [parseFloat(d.lat), parseFloat(d.lng)]);
      //    let paths = [];
      //    let tmp = [];
      //    filterData.forEach((value, index, array) => {
      //       if(index === 0) {
      //          tmp.push(value);
      //       } else {
      //          let spacedist = Math.sqrt((value.lat-array[index-1].lat)**2 + (value.lng-array[index-1].lng)**2);
      //          let timedist = value.timestamp - array[index-1].timestamp;
      //          if(spacedist <= 0.005) {
      //             tmp.push(value)
      //          } else {
      //             tmp = [];
      //          }
      //       }
      //    })
         
      //    // zoom the map to the polyline
      //    // this.map.fitBounds(polyline.getBounds());
      // }
   }
}
</script>

<style scoped>
@import "~leaflet/dist/leaflet.css";
#mapid {
   height: 100%;
}
</style>