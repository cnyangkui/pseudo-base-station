<template>
   <div id="mapid"></div>
</template>

<script>
import L from 'leaflet';
// import mapProvider from '../assets/script/leaflet.MapProviders.js'
// import icon from 'leaflet/dist/images/marker-icon.png';
// import iconShadow from 'leaflet/dist/images/marker-shadow.png';
export default {
   props: ['defaultData', 'datasetWithTime', 'isplay', 'replay', 'visway', 'persent'],
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
         myPersent: 0,
         status: 0, //0:第一次播放, 1:正常播放, 2：拖拽时间轴后播放
      }
   },
   created: function () {
      this.$root.eventHub.$on('change-timeline', this.changeTimeline)
   },
   // 最好在组件销毁前
   // 清除事件监听
   beforeDestroy: function () {
      this.$root.eventHub.$off('change-timeline', this.changeTimeline)
   },
   mounted() {
      this.$nextTick(() => {
         // let mymap = L.map('mapid').setView([39.9788, 116.30226], 13);
         // L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         //    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
         // }).addTo(mymap);
         console.log('该天数据量大小：' + this.defaultData.length)
         this.initMap();
         this.addMapLayer();
         this.addGroupLayer();
         this.clearMarkers();
         this.addMapLayersToLayer(this.defaultData);
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
      // 根据原始数据，创建marker添加到新数组中
      addMarkers(dataset) {
         let markers = [];
         dataset.forEach(d => {
            let arr = d.split('\t');
            let date = arr[0];
            let longitude = parseFloat(arr[1].split(',')[0]);
            let latitude = parseFloat(arr[1].split(',')[1]);
            let marker = L.circle([latitude, longitude], {
               color: 'blue',
               opacity: 0.05,
               radius: 50
            });
            markers.push(marker);
            // this.groupLayer.addLayer(marker); 
         })
         return markers;
      },
      // 将marker添加到layer
      addMarkersDirectToLayer(markers) {
         markers.forEach(d => {
            this.groupLayer.addLayer(d);
         })
      },
      // 直接根据原始数据创建marker，添加到markers中，再添加入layer中
      addMapLayersToLayer(dataset) {
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
         }, 0.01);
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
         }, 0.015);
         this.deleteIntervals.push(deleteInterval);
         // console.log(deleteInterval);
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
      // 播放
      play(way) {
         let flag = true;
         while(flag) {
            if(this.datasetWithTime !== null && this.datasetWithTime !== undefined) {
               flag = false;
            }
         }
         let startIndex = 0;
         if(this.status === 0) { //第一次播放 this.markers.length === this.defaultData.length
            // 第一次Play，清空原始所有marker，开始播放
            this.status = 1;
            this.clearMarkers();
            this.addMarkersWithTimeline(this.datasetWithTime, startIndex);
            if(way === '1') {
               // console.log('addMarkers, deletedMarkers:' + startIndex + ',' + this.deletedMarkers.length);
               // 一段时间后开始删除点
               setTimeout(this.deleteMarkersWithTimeline, 10000, this.datasetWithTime, this.deletedMarkers.length);
            }
         } else if(this.status === 1) { // 正常播放
            // 正常播放，不需要清除点, 立即添加或删除
            startIndex = this.markers.length;
            this.addMarkersWithTimeline(this.datasetWithTime, startIndex);
            if(way === '1') {
               console.log('addMarkers, deletedMarkers:' + startIndex + ',' + this.deletedMarkers.length);
               this.deleteMarkersWithTimeline(this.datasetWithTime, this.deletedMarkers.length);
            }
         } else if(this.status === 2) { //拖动timeline
            // 拖动timeline, 如果是way2, 5s后开始删除点
            this.status = 1;
            startIndex = this.markers.length;
            this.addMarkersWithTimeline(this.datasetWithTime, startIndex);
            if(way === '1') {
               // console.log('addMarkers, deletedMarkers:' + startIndex + ',' + this.deletedMarkers.length);
               setTimeout(this.deleteMarkersWithTimeline, 10000, this.datasetWithTime, this.deletedMarkers.length);
            }
         }
      },
      // 暂停
      pause(way) {
         // 清空所有interval
         this.clearIntervals();
      },
      // 重播
      rePlay(way) {
         // 清空所有点和interval，开始播放
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
            setTimeout(this.deleteMarkersWithTimeline, 5000, this.datasetWithTime, 0);
         }
      },
      // 清除所有interval
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
      },
      // 拖拽时间轴
      changeTimeline(val) {
         this.pause(this.visway); //暂停
         this.clearMarkers();
         let startIndex = Math.floor(parseFloat(val.persent/100) * this.datasetWithTime.length);
         console.log('changeTimeline: ' + startIndex + ',' + parseFloat(val.persent/100));
         if(this.visway === '0') {
            this.addMapLayersToLayer(this.datasetWithTime.slice(0, startIndex));
            // this.myPersent = val.persent;
         } else if(this.visway === '1') {
            this.markers = this.addMarkers(this.datasetWithTime.slice(0, startIndex));
            this.deletedMarkers = this.addMarkers(this.datasetWithTime.slice(0, startIndex));
         }
         this.status = 2; // 置于推拽状态
         if(this.isplay) {
            this.play(this.visway); //播放
         }
      }
   },
   watch: {
      defaultData: function(newV, oldV) {
         // console.log(newV.length);
         this.clearMarkers();
         this.addMapLayersToLayer(newV);
      },
      datasetWithTime: function(newV, oldV) {
         // console.log(newV.length)
         this.status = 0;
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
         this.addMapLayersToLayer(this.defaultData);
         this.clearIntervals();
         this.status = 0;
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
      },
      // 百分比变化，更新时间轴值
      myPersent: function(newV, oldV) {
         console.log('mypresent:', newV);
         this.$root.eventHub.$emit('timeline-changes', {'persent': parseFloat(newV)});
      },
      // markers变化，更新curtime
      markers: function(newV, oldV) {
         console.log('update marsers............')
         // 第一次播放，使时间轴值为0
         if(this.status === 0) {
            console.log('first play....')
            this.myPersent = 0;
            if(this.datasetWithTime) {
               this.$root.eventHub.$emit('curtime-changes', {'curtime': this.datasetWithTime[0].split('\t')[0] || ''});
            }
         } else { //newV.length !== this.datasetWithTime.length
            // 计算百分比，更新时间轴值和curtime
            let result = newV.length / this.datasetWithTime.length * 100;
            this.myPersent = result.toFixed(2);
            this.$root.eventHub.$emit('curtime-changes', {'curtime': this.datasetWithTime[newV.length].split('\t')[0] || ''});
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