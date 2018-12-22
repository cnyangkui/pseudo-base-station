<template>
    <div id="heatmap">
    </div>
</template>

<script>
import * as d3 from 'd3';
export default {
    props: ["dataset"],
    data() {
        return {
            startdate: null,
            enddate: null,
            daynum: null,
            datelist: [],
            starttime: null,
            endtime: null,
            timelist: [],
            minV: null,
            maxV: null,
            heatmap: {
                width: 420,
                height: 225,
                padding: {paddingLeft: 50, paddingRight: 5, paddingTop: 25, paddingBottom: 10},
                svg: null,
            }
        }
    },
    mounted() {
        this.$nextTick(() => {
            this.dataPreprocessing();
            this.setHeatmapSize('#heatmap', this.heatmap.width, this.heatmap.height);
            this.render();
        })
    },
    methods: {
        render() {
            this.svg = d3
                .select('#heatmap')
                .append('svg')
                .attr('width', this.heatmap.width)
                .attr('height', this.heatmap.height);

            let rectsize = Math.min((this.heatmap.width-this.heatmap.padding.paddingLeft-this.heatmap.padding.paddingRight)/(this.endtime-this.starttime), (this.heatmap.height-this.heatmap.padding.paddingTop-this.heatmap.padding.paddingBottom)/this.daynum);
            let rowTitle = this.svg
                .selectAll('text.rowtitle')
                .data(this.timelist)
                .enter()
                .append('text')
                .attr('class', 'rowtitle')
                .attr('x', (d, i) => {
                    return rectsize * i;
                })
                .attr('y', 0)
                .attr('transform', (d, i) => {
                    return `translate(${this.heatmap.padding.paddingLeft}, ${this.heatmap.padding.paddingTop})`;
                })
                .text((d, i) => {
                    if(i % 4 == 0) {
                        return i * (24 / this.timelist.length).toFixed(1);
                    } else{
                        return '';
                    }
                })
                .style('font-size', 8);

            let colTitle = this.svg
                .selectAll('text.coltitle')
                .data(this.datelist)
                .enter()
                .append('text')
                .attr('class', 'coltitle')
                .attr('x', (d, i) => {
                    return 0;
                })
                .attr('y', (d, i) => {
                    return i*rectsize;
                })
                .attr('dx', '.2em')
                .attr('dy', '1em')
                .attr('transform', (d, i) => {
                    return `translate(0, ${this.heatmap.padding.paddingTop})`;
                })
                .text((d, i) => {
                    if(i % 4 == 0) {
                        return d;
                    } else{
                        return '';
                    }
                })
                .style('font-size', 8);
            let rects = this.svg
                .selectAll('rect.heat')
                .data(this.dataset)
                .enter()
                .append('rect')
                .attr('class', 'heat')
                .attr('x', (d, i) => {
                    return rectsize * d.time
                })
                .attr('y', (d, i) => {
                    return this.datelist.indexOf(d.date) * rectsize;
                })
                .attr('width', rectsize)
                .attr('height', rectsize)
                .attr('transform', (d, i) => {
                    return `translate(${this.heatmap.padding.paddingLeft}, ${this.heatmap.padding.paddingTop})`;
                })
                .style('fill', (d, i) => {
                    return this.color([0, this.maxV*0.8], ['white', '#409EFF'])(+d.count);
                })
                .style('cursor', 'pointer')
                .style('opacity', 0.8);

            let tooltip = this.svg.append('text')
                .attr('x', this.heatmap.padding.paddingLeft)
                .attr('y', this.heatmap.height)
                .text('aaa')
                .style('font-size', 12)
                .style('fill', 'red')
                .style('display', 'none');
       
            d3.selectAll('rect.heat')
                .on('mouseover', (d, i) => {
                    tooltip
                      .style('display', '')
                      .text(`${d.date} ${d.time*(24 / this.timelist.length).toFixed(1)}h-${(d.time+1)*(24 / this.timelist.length).toFixed(1)}h ${d.count}`);
                })
                .on('mouseout', (d, i) => {
                    tooltip
                      .style('display', 'none');
                })
        },
        setHeatmapSize(selection, width, height) {
            d3
              .select(selection)
              .attr('width', width)
              .attr('height', height)
        },
        color(valueExtent, colorExtent) {
            return d3.scaleLinear()
                .domain(valueExtent)
                .range(colorExtent);
        },
        dataPreprocessing() {
            let daylist = Array.from(new Set(this.dataset.map(d => d.date)));
            daylist.sort();
            this.startdate = daylist[0];
            this.enddate = daylist[daylist.length - 1];
            this.daynum = this.computeDayDist(this.startdate, this.enddate) + 1;
            [this.starttime, this.endtime] = d3.extent(this.dataset.map(d => d.time));
            [this.minV, this.maxV] = d3.extent(this.dataset.map(d => d.count));
            for(let i=this.starttime; i<=this.endtime; i++) {
                this.timelist.push(i)
            }
            for(let i=0; i<this.daynum; i++) {
                this.datelist.push(this.addDay(this.startdate, i));
            }
        },
        computeDayDist(startTime, endTime) {
            let start = new Date(startTime).getTime();
            let end = new Date(endTime).getTime();
            return Math.floor((end-start) / (24*60*60*1000));
        },
        addDay(startTime, daynum) {
            let start = new Date(startTime).getTime();
            let end = start + daynum * 24*60*60*1000;
            let date = new Date(end);
            return this.formatDate(date);
        },
        formatDate(date) {
            let year = date.getFullYear();
            let month = date.getMonth() + 1;
            let day = date.getDate();
            return year + '-' + (month>9?month:'0'+month) + '-' + (day>9?day:'0'+day);
        }
    },
    computed: {

    },
    watch: {

    },
}
</script>
<style scoped>
#heatmap {
    /* width: 420px;
    height: 235px; */
    /* background-color: green; */
    position: absolute;
    z-index: 999;
    right: 25px;
    bottom: 35px;
}
</style>

