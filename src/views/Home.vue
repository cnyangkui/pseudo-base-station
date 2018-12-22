<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
       <el-container>
            <el-aside width="250px">
              <el-menu>
                <el-menu-item-group>
                    <!-- <span slot="title">分组一</span> -->
                    <el-menu-item index="1-1">
                      <div class="block">
                        <span class="demonstration"></span>
                        <el-date-picker
                          v-model="curdate"
                          type="date"
                          placeholder="选择日期"
                          format="yyyy-MM-dd">
                        </el-date-picker>
                      </div>
                    </el-menu-item>
                    <el-menu-item index="1-2">
                      <span style="color:black">Heatmap：  </span>
                      <el-switch
                        v-model="showHeatmap"
                        active-text="可见"
                        inactive-text="隐藏">
                      </el-switch>
                    </el-menu-item>
                    <el-menu-item index="1-3">
                      <!-- <span style="color:black">Play:  </span>
                      <el-switch
                        v-model="play">
                      </el-switch> -->
                      <el-radio v-model="radio" label="0">Way1</el-radio>
                      <el-radio v-model="radio" label="1">Way2</el-radio>
                    </el-menu-item>
                    <el-menu-item index="1-4">
                      <el-button type="primary" size="small" plain @click="play=!play">{{play===true?'Pause':'Play'}}</el-button>
                      <el-button type="primary" size="small" plain @click="replay=!replay;play=true;">Replay</el-button>
                    </el-menu-item>
                  </el-menu-item-group>
                  <el-menu-item-group title="分组2">
                    <el-menu-item index="1-3">选项3</el-menu-item>
                  </el-menu-item-group>
                  <el-submenu index="1-4">
                    <span slot="title">选项4</span>
                    <el-menu-item index="1-4-1">选项1</el-menu-item>
                  </el-submenu>
              </el-menu>
            </el-aside>
            <el-main>
              <LeafletMap :dataset="geogWithDateData" :datasetWithTime="geogWithTimeData" :isplay="play" :replay="replay" :visway="radio" v-if="flag"></LeafletMap>
              <Heatmap :dataset="timeCountData" v-if="flag" v-show="showHeatmap"></Heatmap>
            </el-main>
        </el-container>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import LeafletMap from '@/components/LeafletMap'
import Heatmap from '@/components/Heatmap'
import axios from 'axios'
export default {
  name: 'home',
  data() {
    return {
      allGeogWithDateData: null,
      allGeogWithTimeData: null,
      geogWithDateData: null,
      geogWithTimeData: null,
      flag: false,
      timeCountData: null,
      flag2: false,
      curdate: '2017-04-01',
      showHeatmap: false,
      play: false,
      replay: false,
      radio: '0',
    }
  },
  components: {
    LeafletMap,
    Heatmap
  },
  created: function() {
    this.$root.eventHub.$on('re-pause', this.rePause)
  },
  mounted: function() {
    this.$nextTick(() => {
      axios.get('/static/data/geog.txt')
        .then((response) => {
          this.allGeogWithDateData = response.data.split('\n');
          console.log('original, date:' + this.allGeogWithDateData.length)
          this.geogWithDateData = this.allGeogWithDateData.filter(d => d.startsWith(this.curdate))
          this.flag = true
        })
        .catch((error) => {
          console.log(error);
        });
      axios.get('/static/data/geogTime.txt')
          .then((response) => {
            this.allGeogWithTimeData = response.data.split('\n');
            console.log('original, time:' + this.allGeogWithTimeData.length)
            this.geogWithTimeData = this.allGeogWithTimeData.filter(d => d.startsWith(this.curdate))
            console.log('geogWithTimeData:' + this.geogWithTimeData.length);
          })
          .catch((error) => {
            console.log(error);
          });
      axios.get('/static/data/timecount.txt')
        .then((response) => {
          let alldata = response.data.split('\n');
          this.timeCountData = []
          alldata.forEach(d => {
            if(d !== '') {
              let arr = d.split('\t');
              let arr2 = arr[0].split(',');
              this.timeCountData.push({'date': arr2[0], 'time': parseInt(arr2[1]), 'count': parseInt(arr[1])});
            }
          })
          this.flag2 = true;
        })
        .catch((error) => {
          console.log(error);
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
    rePause() {
      
    }
  }
}
</script>
<style scoped>
.home, .el-container {
  height: 100%;
}
</style>

