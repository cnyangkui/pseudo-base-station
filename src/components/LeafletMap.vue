<template>
   <div id="mapid"></div>
</template>

<script>
import L from 'leaflet';
// import mapProvider from '../assets/script/leaflet.MapProviders.js'
// import icon from 'leaflet/dist/images/marker-icon.png';
// import iconShadow from 'leaflet/dist/images/marker-shadow.png';
export default {
   props: ['dataset', 'datasetWithTime', 'play', 'pause'],
   data() {
      return {
         map: null,
         markers: [],
         tileLayer: null,
         groupLayer: null,
         mapConfig: {
            zoom: 10,
            center: [39.92, 116.46],
            minZoom: 3,
            maxZoom: 18,
         },
         addInterval: null,
         deleteInterval: null,
      }
   },
   mounted() {
      this.$nextTick(() => {
         // let mymap = L.map('mapid').setView([39.9788, 116.30226], 13);
         // L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         //    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
         // }).addTo(mymap);
         console.log(this.dataset.length)
         this.initMap();
         this.addMapLayer();
         this.addGroupLayer();
         this.clearMarkers();
         this.addMarkers(this.dataset);
      })
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
            let arr = d.split('\t');
            let date = arr[0];
            let longitude = parseFloat(arr[1].split(',')[0]);
            let latitude = parseFloat(arr[1].split(',')[1]);
            // console.log(longitude, latitude)
            // L.marker([longitude, latitude]).addTo(this.map);
            // L.circle([latitude, longitude], {
            //    color: 'blue',
            //    // fillColor: 'red',
            //    opacity: 0.05,
            //    radius: 50
            // }).addTo(this.map);
            let marker = L.circle([latitude, longitude], {
               color: 'blue',
               opacity: 0.05,
               radius: 50
            });
            this.markers.push(marker);
            this.groupLayer.addLayer(marker); 
         })
      },
      clearMarkers() {
         this.groupLayer.clearLayers();
         this.markers = [];
      },
      markersWithTimeline(dataset) {
         let size = dataset.length;
         console.log('size:' + size);
         let groupLayer = this.groupLayer;
         let markers = this.markers;
         // setTimeout(function() {
         //    let i = 0;
         //    deleteInterval = setInterval(function() {
         //       if(i === size) {
         //          clearInterval(deleteInterval);
         //          return;
         //       }
         //       //do whatever here...
         //       groupLayer.removeLayer(markers[i]);
         //       i = i + 1;
         //    }, 1);
         //    this.deleteInterval = deleteInterval;
         //    console.log('delete:' + this.deleteInterval)
         // }, 3000);


         // let i = 0;
         // let sleep = this.sleep;
         // let deleteInterval = setInterval(function() {
         //    if(i === 0) {
         //       // console.log(sleep)
         //       console.log('start sleep...')
         //       sleep(3000);
         //    }
         //    console.log('end sleep...')
         //    if(i === size) {
         //       clearInterval(deleteInterval);
         //       return;
         //    }
         //    //do whatever here...
         //    groupLayer.removeLayer(markers[i]);
         //    i = i + 1;
         // }, 1);
         // this.deleteInterval = deleteInterval;
         // console.log('delete:' + this.deleteInterval)


         let j = 0;
         let addInterval = setInterval(function() {
            if(j === size) {
               clearInterval(addInterval);
               return;
            }
            //do whatever here...
            let d = dataset[j];
            let arr = d.split('\t');
            let time = arr[0];
            let longitude = parseFloat(arr[1].split(',')[0]);
            let latitude = parseFloat(arr[1].split(',')[1]);
            let marker = L.circle([latitude, longitude], {
               color: 'blue',
               opacity: 0.05,
               radius: 50
            });
            markers.push(marker);
            this.markers = markers;
            // console.log(this.markers.length);
            groupLayer.addLayer(marker); 
            this.groupLayer = groupLayer;
            j = j + 1;
         }, 1);
         this.addInterval = addInterval;
         // console.log(addInterval, this.addInterval)
      },
      sleep(n) {
         let start = new Date().getTime();
         let flag = true
         //  console.log('休眠前：' + start);
         while (flag) {
            if (new Date().getTime() - start > n) {
               console.log('dddddddddd')
               flag = false;
            }
         }
         console.log('end')
      }
   },
   watch: {
      dataset: function(newV, oldV) {
         console.log(newV.length);
         this.clearMarkers();
         this.addMarkers(newV);
      },
      play: function(newV, oldV) {
         console.log(newV)
         if(newV) {
            this.clearMarkers();
            let flag = true;
            while(flag) {
               if(this.datasetWithTime !== null && this.datasetWithTime !== undefined) {
                  flag = false;
               }
            }
            this.markersWithTimeline(this.datasetWithTime);
         }
      },
      pause: function(newV, oldV) {
         if(newV) {
            console.log('pause:' + newV);
            console.log('pause add:' + this.addInterval)
            console.log('pause delete:' + this.deleteInterval)
            clearInterval(this.addInterval);
            // clearInterval(this.deleteInterval);
            // this.clearMarkers();
         }
      }
   },
   computed: {
   
   }
}
</script>


<style scoped>
@import "~leaflet/dist/leaflet.css";
#mapid {
   height: 100%;
}
</style>