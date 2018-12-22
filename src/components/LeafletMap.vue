<template>
   <div id="mapid"></div>
</template>

<script>
import L from 'leaflet';
// import mapProvider from '../assets/script/leaflet.MapProviders.js'
// import icon from 'leaflet/dist/images/marker-icon.png';
// import iconShadow from 'leaflet/dist/images/marker-shadow.png';
export default {
   props: ['dataset', 'datasetWithTime', 'isplay', 'replay', 'visway'],
   data() {
      return {
         map: null,
         markers: [],
         deletedMarkers: [],
         tileLayer: null,
         groupLayer: null,
         mapConfig: {
            zoom: 10,
            center: [39.92, 116.46],
            minZoom: 3,
            maxZoom: 18,
         },
         addIntervals: [],
         deleteIntervals: [],
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
      addMarkersDirect(markers) {
         markers.forEach(d => {
            this.groupLayer.addLayer(d);
         })
      },
      clearMarkers() {
         this.groupLayer.clearLayers();
         this.markers = [];
         this.deletedMarkers = [];
      },
      addMarkersWithTimeline(dataset, startIndex) {
         let size = dataset.length;
         let groupLayer = this.groupLayer;
         let markers = this.markers;

         let addInterval = setInterval(function() {
            if(startIndex === size) {
               clearInterval(addInterval);
               return;
            }
            //do whatever here...
            let d = dataset[startIndex];
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
            groupLayer.addLayer(marker); 
            this.markers = markers;
            this.groupLayer = groupLayer;
            startIndex = startIndex + 1;
         }, 0.1);
         this.addIntervals.push(addInterval);
      },
      deleteMarkersWithTimeline(dataset, startIndex) {
         let size = dataset.length;
         let groupLayer = this.groupLayer;
         let markers = this.markers;
         let deletedMarkers = this.deletedMarkers;

         let deleteInterval = setInterval(function() {
            if(startIndex === size) {
               clearInterval(deleteInterval);
               return;
            }
            //do whatever here...
            if(deletedMarkers.length < markers.length) {
               groupLayer.removeLayer(markers[startIndex]);
               deletedMarkers.push(markers[startIndex]);
               this.deletedMarders = deletedMarkers;
               this.groupLayer = groupLayer;
               startIndex = startIndex + 1;
            }
         }, 0.15);
         this.deleteIntervals.push(deleteInterval);
         console.log(deleteInterval);
      },
      sleep(n) {
         let start = new Date().getTime();
         let flag = true
         //  console.log('休眠前：' + start);
         while (flag) {
            if (new Date().getTime() - start > n) {
               flag = false;
            }
         }
      },
      play(way) {
         let flag = true;
         while(flag) {
            if(this.datasetWithTime !== null && this.datasetWithTime !== undefined) {
               flag = false;
            }
         }
         // this.clearMarkers();
         // this.addMarkersDirect(this.markers);
         let startIndex = 0;
         if(this.markers.length === this.dataset.length) {
            // 第一次Play
            this.clearMarkers();
            this.addMarkersWithTimeline(this.datasetWithTime, startIndex);
            if(way === '1') {
               console.log('addMarkers, deletedMarkers:' + startIndex + ',' + this.deletedMarkers.length);
               setTimeout(this.deleteMarkersWithTimeline, 5000, this.datasetWithTime, this.deletedMarkers.length);
            }
         } else {
            startIndex = this.markers.length;
            this.addMarkersWithTimeline(this.datasetWithTime, startIndex);
            if(way === '1') {
               console.log('addMarkers, deletedMarkers:' + startIndex + ',' + this.deletedMarkers.length);
               this.deleteMarkersWithTimeline(this.datasetWithTime, this.deletedMarkers.length);
            }
         }
      },
      pause(way) {
         // console.log(this.addInterval, this.deleteInterval);
         // if(way === '1') {
         //    if(this.deleteInterval) {
         //       clearInterval(this.deleteInterval);
         //    } else {
         //       alert("请稍后再试...");
         //       this.$root.eventHub.$emit('re-pause', '');
         //       return;
         //    }
         // }
         // clearInterval(this.addInterval);
         // this.addInterval = null;
         // this.deleteInterval = null;
         this.clearIntervals();
         
      },
      rePlay(way) {
         console.log('replay。。。。。。')
         this.clearMarkers();
         this.clearIntervals();
         let flag = true;
         while(flag) {
            if(this.datasetWithTime !== null && this.datasetWithTime !== undefined) {
               flag = false;
            }
         }
         this.addMarkersWithTimeline(this.datasetWithTime, 0);
         if(way === '1') {
            console.log('start delete markers...');
            setTimeout(this.deleteMarkersWithTimeline, 5000, this.datasetWithTime, 0);
         }
      },
      clearIntervals() {
         console.log('interval:' + this.addIntervals + '|' + this.deleteIntervals);
         if(this.addIntervals) {
            this.addIntervals.forEach(d => {
               clearInterval(d);
            })
         }
         if(this.deleteIntervals) {
            this.deleteIntervals.forEach(d => {
               clearInterval(d);
            })
         }
         this.addIntervals = [];
         this.deleteIntervals = [];
      }
   },
   watch: {
      dataset: function(newV, oldV) {
         console.log(newV.length);
         this.clearMarkers();
         this.addMarkers(newV);
      },
      isplay: function(newV, oldV) {
         console.log('isplay:' + newV)
         if(newV) {
            this.play(this.visway);
         } else {
            this.pause(this.visway);
         }
      },
      replay: function(newV, oldV) {
         console.log('replay:' + newV);
         this.rePlay(this.visway);
      },
      visway: function(newV, oldV) {
         this.clearMarkers();
         this.addMarkers(this.dataset);
         this.clearIntervals();
      },
      deleteIntervals: function(newV, oldV) {
         if(this.isplay === false && newV.length !==0) {
            this.deleteIntervals.forEach(d => {
               clearInterval(d);
            })
         } else if(this.isplay === true && newV.length > 1) {
            this.deleteIntervals.forEach((d, i) => {
               if (i < newV.length-1) {
                  clearInterval(d);
               }
            })
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