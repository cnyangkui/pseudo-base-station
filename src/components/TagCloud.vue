<template>
    <div id="tagcloud">
    </div>
</template>
<script>
import * as d3 from 'd3';
import * as cloud from 'd3.layout.cloud'
export default {
    props: ['defaultData'],
    data() {
        return {
            tagCloud: {
                width: '100%',
                height: '100%',
                padding: {top: 30, bottom: 30, left: 30, right: 30},
                svg: null,
                fontsize: [10, 100],
                size: [900, 600],
                // domain: [0, 800]
            }
        }
    },
    mounted() {
      this.$nextTick(() => {
        let data = this.dataPreprocessing();
        this.setChartSize('#tagcloud', this.tagCloud.width, this.tagCloud.height);
        this.render(data);
      })
    },
    methods: {
        dataPreprocessing() {
            let data = this.defaultData.sort((a, b) => b.count - a.count)
            data = data.slice(0, 200)
            let extent = d3.extent(data, d => d.count)
            console.log(extent)
            let scale = this.scale(this.tagCloud.fontsize, extent);
            return data.map(d => {
                return {'word': d.word, 'size':scale(d.count)}
            })
        },
        setChartSize(selection, width, height) {
            d3
              .select(selection)
              .style('width', width)
              .style('height', height)
        },
        color() {
            return d3.scaleOrdinal(d3.schemeCategory10);
        },
        scale(range, domain) {
            return d3.scaleLinear()
                .range(range)
                .domain(domain);
        },
        render(wordsdata) {
            this.svg = d3
                .select('#tagcloud')
                .append('svg')
                .attr('width', this.tagCloud.width)
                .attr('height', this.tagCloud.height);
            
            let mainG = this.svg
                .append('g').attr('class', 'mainG')
                .attr('transform', `translate(${this.tagCloud.padding.left}, ${this.tagCloud.padding.top})`)
            
            let fill = this.color();

            let layout = cloud()
                .size(this.tagCloud.size)
                .words(wordsdata)
                .padding(5)
                // .rotate(() => {
                //     return ~~(Math.random() * 2) * 90;
                // })
                .rotate(0)
                .font("Impact")
                .fontSize((d) => d.size)
                .on("end", draw);

            layout.start();

            function draw(words) {
                mainG
                    .append("g")
                    .attr("transform", `translate(${layout.size()[0]/2}, ${layout.size()[1]/2})`)
                    .selectAll("text")
                    .data(words)
                    .enter()
                    .append("text")
                    .style("font-size", d => d.size + "px")
                    .style("font-family", "Impact")
                    .style("fill", (d, i) => fill(i))
                    .attr("text-anchor", "middle")
                    .attr("transform", (d) => `translate(${d.x}, ${d.y})rotate(${d.rotate})`)
                    .text(d => d.word);
            }
        }

    }
}
</script>
<style scoped>
#tagcloud {
    /* background-color: red; */
}
</style>

