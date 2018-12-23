<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
       <el-container>
            <el-aside>
              <el-menu @open="handleOpen" @close="handleClose" @select="handleSelect" default-active="1-1" :unique-opened="true">
                <el-submenu index="1" collapse="isCollapse1">
                  <template slot="title">
                    <i class="el-icon-location"></i>
                    <span>Map</span>
                  </template>
                  <!-- <el-menu-item-group> -->
                    <!-- <span slot="title">分组一</span> -->
                    <el-menu-item index="1-1">
                        <!-- <span class="demonstration"></span> -->
                        <el-date-picker
                          v-model="leftletMap.curdate"
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
                  <!-- </el-menu-item-group> -->
                </el-submenu>
                <el-menu-item index="2">
                  <i class="el-icon-menu"></i>
                  <span slot="title">TagCloud</span>
                </el-menu-item>
                <el-submenu index="3">
                  <template slot="title">
                    <i class="el-icon-location"></i>
                    <span>xxx</span>
                  </template>
                  <el-menu-item-group title="分组2">
                    <el-menu-item index="2-1">选项3</el-menu-item>
                  </el-menu-item-group>
                  <el-submenu index="2-2">
                    <span slot="title">选项4</span>
                    <el-menu-item index="2-2-1">选项1</el-menu-item>
                  </el-submenu>
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
        </el-container>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import LeafletMap from '@/components/LeafletMap'
import Heatmap from '@/components/Heatmap'
import TagCloud from '@/components/TagCloud'

import axios from 'axios'
export default {
  name: 'home',
  data() {
    return {
      leftletMap: {
        allGeogWithDateData: null,
        allGeogWithTimeData: null,
        geogWithDateData: null,
        geogWithTimeData: null,
        flag: false,
        visway: '0',
        curdate: '2017-04-01',
        play: false,
        replay: false,
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
      module: '1',
      
    }
  },
  components: {
    LeafletMap,
    Heatmap,
    TagCloud
  },
  created: function() {
    this.$root.eventHub.$on('re-pause', this.rePause)
  },
  mounted: function() {
    this.$nextTick(() => {
      axios.get('/static/data/geog.txt')
        .then((response) => {
          this.leftletMap.allGeogWithDateData = response.data.split('\n');
          console.log('original, date:' + this.leftletMap.allGeogWithDateData.length)
          this.leftletMap.geogWithDateData = this.leftletMap.allGeogWithDateData.filter(d => d.startsWith(this.leftletMap.curdate))
          this.leftletMap.flag = true
        })
        .catch((error) => {
          console.error(error);
        });
      axios.get('/static/data/geogTime.txt')
          .then((response) => {
            this.leftletMap.allGeogWithTimeData = response.data.split('\n');
            console.log('original, time:' + this.leftletMap.allGeogWithTimeData.length)
            this.leftletMap.geogWithTimeData = this.leftletMap.allGeogWithTimeData.filter(d => d.startsWith(this.leftletMap.curdate))
            console.log('geogWithTimeData:' + this.leftletMap.geogWithTimeData.length);
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
    this.$root.eventHub.$off('re-pause', this.rePause)
  },
  computed: {
    
  },
  watch: {
    curdate: function(newV, oldV) {
      // this.flag = false;
      let year = newV.getFullYear();
      let month = newV.getMonth() + 1;
      let day = newV.getDate();
      let date = year + '-' + (month>9?month:'0'+month) + '-' + (day>9?day:'0'+day);
      this.geogWithDateData = this.allGeogWithDateData.filter(d => d.startsWith(date));
    }
  },
  methods: {
    handleOpen(key, keyPath) {
      console.log(key, keyPath);
      this.module = keyPath[0];
      
    },
    handleClose(key, keyPath) {
      console.log(key, keyPath);
    },
    handleSelect(key, keyPath) {
      console.log(key, keyPath);
      this.module = keyPath[0];
      switch (key) {
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
      
        default:
          break;
      }
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
</style>

