<template>
    <div id="bubble"></div>
</template>

<script>
import * as d3 from 'd3';
export default {
    props: ['defaultData', 'md5text'],
    data() {
        return {
            bubble: {
                width: 0,
                height: 0,
                padding: {top: 0, bottom: 10, left: 10, right: 10},
                svg: null
            }
        }
    },
    mounted() {
        this.$nextTick(() => {
            // console.log(this.md5text)
            let data = this.dataPreprocessing();
            // this.setChartSize('#bubble', this.bubble.width, this.bubble.height);
            this.render(data);
        })
    },
    methods: {
        dataPreprocessing() {
            if(this.defaultData) {
                return this.defaultData.sort((a, b) => b.count - a.count);
            } else {
                return;
            }
        },
        render(dataset) {
            let width = d3.select('#bubble').style('width');
            let height = d3.select('#bubble').style('height');
            this.bubble.width = parseFloat(width.substring(0, width.length-2))
            this.bubble.height = parseFloat(height.substring(0, height.length-2))

            this.svg = d3
                .select('#bubble')
                .append('svg')
                .attr('width', this.bubble.width)
                .attr('height', this.bubble.height);
                
            let color = this.color();

            
            
            let treemap = this.treemap([this.bubble.width, this.bubble.height], 1)
            let root = this.root(dataset);
    
            treemap(root);
    
            let cells = this.svg.selectAll(".cell")
                .data(root.leaves())
                .enter()
                .append("g")
                .attr("class", "cell")
                .attr('transform', `translate(${this.bubble.padding.left}, ${this.bubble.padding.top})`)
                .attr('transform', d => `translate(${d.x0}, ${d.y0})`)
                
            
            cells.append("rect")
                .attr("id", (d, i) => i)
                .attr('width', d => d.x1 - d.x0)
                .attr("height", d => d.y1 - d.y0)
                .attr('fill', 'rgb(224, 201, 136)')
                .style('cursor', 'pointer')
                .on('click', (d, i) => {
                    let md5 = d.data.md5;
                    this.$root.eventHub.$emit('find-point-by-md5', {'md5': md5});
                });

            cells.append("text")
                .attr("font-size", 8)
                .attr("dy", "1em")
                .text(d => d.data.md5.substring(0, 7))
                .attr("font-family", "sans-serif")
                .attr("fill", "white")
                .style('cursor', 'pointer')
                .on('click', (d, i) => {
                    let md5 = d.data.md5;
                    this.$root.eventHub.$emit('find-point-by-md5', {'md5': md5});
                });

            let md5text = Array.from(this.md5text);
            cells.append('title')
                .text(d => {
                    if(md5text != null && md5text != undefined) {
                        let text = md5text.filter(dd => dd.split(',')[0] === d.data.md5)[0];
                        if(text) return text.split(',')[1];
                        else return d.data.md5 + ':' + d.data.count;
                    } else {
                        return d.data.md5 + ':' + d.data.count
                    }
                });
        },
        color() {
            return d3.scaleOrdinal(d3.schemeCategory10);
        },
        treemap(size, padding) {
            return d3.treemap().size(size).padding(padding);
        },
        root(data) {
            return d3.hierarchy({'children': data})
                .sum(d => d.count * 3)
                .sort((a, b) => b.count - a.count);
        }
    }
}
</script>

<style scoped>
#bubble {
    width: 100%;
    height: 100%;
}
</style>

