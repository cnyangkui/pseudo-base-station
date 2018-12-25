<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
       <el-container>
            <el-aside>
              <el-menu @open="handleOpen" @close="handleClose" @select="handleSelect" default-active="1-1" :unique-opened="true">
                <el-submenu index="1">
                  <template slot="title">
                    <i class=""></i>
                    <span>Map</span>
                  </template>
                  <!-- <el-menu-item-group> -->
                    <!-- <span slot="title">分组一</span> -->
                    <el-menu-item index="1-1">
                        <!-- <span class="demonstration"></span> -->
                        <el-date-picker
                          v-model="curdate"
                          type="date"
                          placeholder="选择日期"
                          format="yyyy-MM-dd"
                          size="small"
                          :editable="false"
                          :clearable="false">
                        </el-date-picker>
                    </el-menu-item>
                    <el-menu-item index="1-2">
                      <span style="color:black">Heatmap：  </span>
                      <el-switch
                        v-model="heatmap.show"
                        active-text="可见"
                        inactive-text="隐藏">
                      </el-switch>
                    </el-menu-item>
                    <el-menu-item index="1-3">
                      <!-- <span style="color:black">Play:  </span>
                      <el-switch
                        v-model="play">
                      </el-switch> -->
                      <el-radio v-model="leftletMap.visway" label="0" size="small" :border="true">Way 1</el-radio>
                      <el-radio v-model="leftletMap.visway" label="1" size="small" :border="true">Way 2</el-radio>
                    </el-menu-item>
                    <el-menu-item index="1-4">
                      <el-button type="primary" size="small" plain @click="leftletMap.play=!leftletMap.play">{{leftletMap.play===true?'Pause':'Play'}}</el-button>
                      <el-button type="primary" size="small" plain @click="leftletMap.replay=!leftletMap.replay;leftletMap.play=true;">Replay</el-button>
                    </el-menu-item>
                    <el-menu-item index="1-5">
                      <div class="block">
                        <span class="demonstration">timeline</span>
                        <el-slider v-model="persent" class="slider" :min="0" :max="100" :step="0.01" :format-tooltip="formatTooltip" @change="changeTimeline"></el-slider>
                      </div>
                    </el-menu-item>
                    <el-menu-item index="1-6">
                      <span class="demonstration">current：{{leftletMap.curtime}}</span>
                    </el-menu-item>
                  <!-- </el-menu-item-group> -->
                </el-submenu>
                <el-menu-item index="2">
                  <i class=""></i>
                  <span slot="title">TagCloud</span>
                </el-menu-item>
                <el-submenu index="3">
                  <template slot="title">
                    <i class=""></i>
                    <span>Track</span>
                  </template>
                  <el-menu-item index="3-1">
                    <el-date-picker
                      v-model="curdate2"
                      type="date"
                      placeholder="选择日期"
                      format="yyyy-MM-dd"
                      size="small"
                      :editable="false"
                      :clearable="false">
                    </el-date-picker>
                  </el-menu-item>
                </el-submenu>
              </el-menu>
            </el-aside>
            <el-main v-if="module === '1'">
              <div class="main-content">
                <LeafletMap :defaultData="leftletMap.geogWithDateData" :datasetWithTime="leftletMap.geogWithTimeData" :isplay="leftletMap.play" :replay="leftletMap.replay" :visway="leftletMap.visway" v-if="leftletMap.flag"></LeafletMap>
                <Heatmap :dataset="heatmap.timeCountData" v-if="heatmap.flag" v-show="heatmap.show"></Heatmap>
              </div>
            </el-main>
            <el-main v-else-if="module === '2'">
              <div class="main-content">
                <TagCloud :defaultData="tagCloud.wordcount" v-if="tagCloud.flag"></TagCloud>
              </div>
            </el-main>
            <el-main v-else-if="module === '3'">
              <el-row :gutter="5" class="main-content">
                <el-col :span="18" class="main-content">
                  <TrackInMap :defaultData="trackInMap.trackData"></TrackInMap>
                </el-col>
                <el-col :span="6" class="main-content">
                  <BubbleChart :defaultData="bubbleChart.bubbleData" :md5text="bubbleChart.md5text" v-if="bubbleChart.flag && trackInMap.flag"></BubbleChart>
                </el-col>
              </el-row>
            </el-main>
        </el-container>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import LeafletMap from '@/components/LeafletMap'
import Heatmap from '@/components/Heatmap'
import TagCloud from '@/components/TagCloud'
import TrackInMap from '@/components/TrackInMap'
import BubbleChart from '@/components/BubbleChart'

import axios from 'axios'
import { all } from 'q';
export default {
  name: 'home',
  data() {
    return {
      curdate: '2017-04-01',
      curdate2: '2017-04-01',
      persent: 0,
      leftletMap: {
        allGeogWithDateData: null,
        allGeogWithTimeData: null,
        geogWithDateData: null,
        geogWithTimeData: null,
        flag: false,
        visway: '0',
        play: false,
        replay: false,
        curtime: null,
      },
      heatmap: {
        timeCountData: null,
        flag: false,
        show: false,
      },
      tagCloud: {
        wordcount: null,
        flag: false,
      },
      trackInMap: {
        alldata: null,
        trackData: null,
        flag: false,
      },
      bubbleChart: {
        alldata: null,
        bubbleData: null,
        flag: false,
        md5text: null,
      },
      module: '1',
      
    }
  },
  components: {
    LeafletMap,
    Heatmap,
    TagCloud,
    TrackInMap,
    BubbleChart
  },
  created: function() {
    this.$root.eventHub.$on('timeline-changes', this.timelineChanges);
    this.$root.eventHub.$on('curtime-changes', this.curtimeChanges);
    this.$root.eventHub.$on('find-point-by-md5', this.updateBubbleData);
  },
  mounted: function() {
    this.$nextTick(() => {
      axios.get('/static/data/geog.txt')
        .then((response) => {
          this.leftletMap.allGeogWithDateData = response.data.split('\n');
          console.log('original, date:' + this.leftletMap.allGeogWithDateData.length)
          this.leftletMap.geogWithDateData = this.leftletMap.allGeogWithDateData.filter(d => d.startsWith(this.curdate))
          this.leftletMap.flag = true
        })
        .catch((error) => {
          console.error(error);
        });
      axios.get('/static/data/geogTime.txt')
          .then((response) => {
            this.leftletMap.allGeogWithTimeData = response.data.split('\n');
            console.log('original, time:' + this.leftletMap.allGeogWithTimeData.length)
            this.leftletMap.geogWithTimeData = this.leftletMap.allGeogWithTimeData.filter(d => d.startsWith(this.curdate))
            console.log('geogWithTimeData:' + this.leftletMap.geogWithTimeData.length);
            this.leftletMap.curtime = this.leftletMap.geogWithTimeData[0].split('\t')[0] || '';
          })
          .catch((error) => {
            console.error(error);
          });
      axios.get('/static/data/timecount.txt')
        .then((response) => {
          let alldata = response.data.split('\n');
          this.heatmap.timeCountData = []
          alldata.forEach(d => {
            if(d !== '') {
              let arr = d.split('\t');
              let arr2 = arr[0].split(',');
              this.heatmap.timeCountData.push({'date': arr2[0], 'time': parseInt(arr2[1]), 'count': parseInt(arr[1])});
            }
          })
          this.heatmap.flag = true;
        })
        .catch((error) => {
          console.error(error);
        })
    })

  },
  beforeDestroy: function () {
    this.$root.eventHub.$off('timeline-changes', this.timelineChanges);
    this.$root.eventHub.$off('curtime-changes', this.curtimeChanges);
    this.$root.eventHub.$off('find-point-by-md5', this.updateBubbleData);
  },
  computed: {
    
  },
  watch: {
    curdate: function(newV, oldV) {
      // this.flag = false;
      let date = this.formatTime(newV);
      this.leftletMap.geogWithDateData = this.leftletMap.allGeogWithDateData.filter(d => d.startsWith(date));
    },
    curdate2: function(newV, oldV) {
      this.trackInMap.flag = false;
      this.bubbleChart.flag = false;
      let date = '';
      if(typeof(newV) == 'object') {
        date = this.formatTime(newV)
      } else {
        date = newV;
      }
      let filename = date.replace(/-/g, '') + '.csv';
      axios.get('/static/data/originaldata/' + filename)
        .then((response) => {
          let alldata = response.data.split('\n');
          this.trackInMap.alldata = [];
          this.bubbleChart.md5text = new Set();
          alldata.forEach(d => {
            if(d !== '') {
              let dims = d.split(',');
              if(dims.length === 7) {
                let timestamp = dims[3];
                let tmpdate = this.formatTime(new Date(parseInt(timestamp)));
                if(tmpdate === date) {
                  this.trackInMap.alldata.push({'md5': dims[0], 'phone': dims[2], 'lng': dims[5], 'lat': dims[6]});
                  this.bubbleChart.md5text.add(dims[0] + ',' + dims[1]);
                }
              }
            }
          })
          this.trackInMap.trackData = [];
          this.trackInMap.trackData = this.trackInMap.alldata.filter(d => d.md5 === '');
          this.trackInMap.flag = true;
        })
        .catch((error) => {
          console.error(error);
        });
      axios.get('/static/data/md5datecount.txt')
        .then((response) => {
          let alldata = response.data.split('\n');
          this.bubbleChart.alldata = [];
          alldata.forEach(d => {
            if(d !== '') {
              let arr = d.split('\t');
              let arr2 = arr[0].split(',');
              let count = arr[1];
              let date = arr2[0];
              let md5 = arr2[1];
              this.bubbleChart.alldata.push({'date': date, 'md5': md5, 'count': parseInt(count)});
            }
          })
          this.bubbleChart.bubbleData = this.bubbleChart.alldata.filter(d => d.date === date)
          console.log('bubbleChart:' + this.bubbleChart.bubbleData.length);
          this.bubbleChart.flag = true;
        })
        .catch((error) => {
          console.error(error);
        })
    },
    module: function(newV, oldV) {
      switch (newV) {
        case '2':
          if(this.tagCloud.flag === false) {
            axios.get('/static/data/wordcount.txt')
              .then((response) => {
                let alldata = response.data.split('\n');
                this.tagCloud.wordcount = [];
                alldata.forEach(d => {
                  if(d !== '') {
                    let arr = d.split('\t');
                    this.tagCloud.wordcount.push({'word': arr[0], 'count': parseInt(arr[1])});
                  }
                })
                console.log('wordcount:' + this.tagCloud.wordcount.length);
                this.tagCloud.flag = true;
              })
              .catch((error) => {
                console.error(error);
              })
          }
          break;
        case '3':
          if(this.trackInMap.flag === false) {
            let date = '';
            if(typeof(this.curdate2) == 'object') {
              date = this.formatTime(this.curdate)
            } else {
              date = this.curdate2;
            }
            let filename = date.replace(/-/g, '') + '.csv';
            console.log(filename)
            axios.get('/static/data/originaldata/' + filename)
              .then((response) => {
                let alldata = response.data.split('\n');
                this.trackInMap.alldata = [];
                this.bubbleChart.md5text = new Set();
                alldata.forEach(d => {
                  if(d !== '') {
                    let dims = d.split(',');
                    if(dims.length === 7) {
                      let timestamp = dims[3];
                      let tmpdate = this.formatTime(new Date(parseInt(timestamp)));
                      if(tmpdate === date) {
                        this.trackInMap.alldata.push({'md5': dims[0], 'phone': dims[2], 'lng': dims[5], 'lat': dims[6]});
                        this.bubbleChart.md5text.add(dims[0] + ',' + dims[1]);
                      }
                    }
                  }
                })
                this.trackInMap.trackData = [];
                this.trackInMap.trackData = this.trackInMap.alldata.filter(d => d.md5 === '');
                this.trackInMap.flag = true;
              })
              .catch((error) => {
                console.error(error);
              })
          }
          if(this.bubbleChart.flag === false) {
            let date = '';
            if(typeof(this.curdate2) == 'object') {
              date = this.formatTime(this.curdate)
            } else {
              date = this.curdate2;
            }
            axios.get('/static/data/md5datecount.txt')
              .then((response) => {
                let alldata = response.data.split('\n');
                this.bubbleChart.alldata = [];
                alldata.forEach(d => {
                  if(d !== '') {
                    let arr = d.split('\t');
                    let arr2 = arr[0].split(',');
                    let count = arr[1];
                    let date = arr2[0];
                    let md5 = arr2[1];
                    this.bubbleChart.alldata.push({'date': date, 'md5': md5, 'count': parseInt(count)});
                  }
                })
                this.bubbleChart.bubbleData = this.bubbleChart.alldata.filter(d => d.date === date)
                console.log('bubbleChart:' + this.bubbleChart.bubbleData.length);
                this.bubbleChart.flag = true;
              })
              .catch((error) => {
                console.error(error);
              })
          }
          break;
      
        default:
          break;
      }
    }
  },
  methods: {
    formatTime(date) {
      let year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDate();
      return year + '-' + (month>9?month:'0'+month) + '-' + (day>9?day:'0'+day);
    },
    updateBubbleData(val) {
      this.trackInMap.trackData = [];
      if(this.trackInMap.flag) {
        let curdate = this.curdate2;
        if(typeof(curdate) === 'object') {
          curdate = this.formatTime(curdate);
        }
        this.trackInMap.trackData = this.trackInMap.alldata.filter(d => d.md5 === val.md5);
      }
    },
    changeTimeline(val) {
      this.persent = val;
      this.$root.eventHub.$emit('change-timeline', {'persent': this.persent})
    },
    curtimeChanges(val) {
      this.leftletMap.curtime = val.curtime;
    },
    timelineChanges(val) {
      // console.log('timelinechanges:' + typeof(val.persent) + ',' + val.curtime)
      this.persent = val.persent;
      // this.persent = val;
    },
    handleOpen(key, keyPath) {
      // console.log(key, keyPath);
      if(keyPath) {
        this.module = keyPath[0];
        console.log(this.module)
      }
    },
    handleClose(key, keyPath) {
      // console.log(key, keyPath);
      if(keyPath) {
        this.module = keyPath[0];
        console.log(this.module)
      }
    },
    handleSelect(key, keyPath) {
      // console.log(key, keyPath);
      if(keyPath) {
        this.module = keyPath[0];
        console.log(this.module)
      }
    },
    formatTooltip(val) {
      return val + '%';
    }
  }
}
</script>
<style scoped>
.home, .el-container, .el-menu {
  height: 100%;
}
.el-date-editor.el-input, .el-date-editor.el-input__inner {
  width: 150px;
}
.main-content {
  height: 100%;
}
.block {
  display: 'block';
}
.slider {
  width: 65%;
  float: right;
  margin-top: 6px;
  margin-right: 0px;
}
</style>

