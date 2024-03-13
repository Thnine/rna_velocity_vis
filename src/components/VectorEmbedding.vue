<template>

    <div>
        <div class="vector-embedding-container">
            <svg class="vector-embedding-canvas" ref="vector-embedding-canvas">

            </svg>
        </div>

    </div>

</template>

<script>
import * as d3 from 'd3'

export default {
    name:'VectorEmbedding',
    props:['data'],

    data(){
        return {
            //配置项
            padding:{
                'top':60,
                'left':20,
                'right':20,
                'bottom':20,
            }
        }
    },



    methods:{
        redraw(){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            svg.selectAll('*').remove()

            const width = svg.node().getBoundingClientRect().width - self.padding.left - self.padding.right;
            const height = svg.node().getBoundingClientRect().height - self.padding.top - self.padding.bottom;

            const title = JSON.parse(JSON.stringify(self.data['title']))
            let color_scheme  = JSON.parse(JSON.stringify(self.data['color_scheme']))
            let points = JSON.parse(JSON.stringify(self.data['points']))

            const scatter = svg.append('g')
            const legend = svg.append('g')

            //title
            svg.append('text')
                .text(title)
                .attr('x',0.5 * width + this.padding.left)
                .attr('y',30)
                .style('font-size',20)

            //scatter
            //scale
            let minX = Math.min(...points.map(d=>d.x));
            let maxX = Math.max(...points.map(d=>d.x));
            let minY = Math.min(...points.map(d=>d.y));
            let maxY = Math.max(...points.map(d=>d.y));
            const posXScale = d3
                .scaleLinear()
                .domain([minX, maxX])
                .range([self.padding.left, self.padding.left + width]);
            const posYScale = d3
                .scaleLinear()
                .domain([minY, maxY])
                .range([self.padding.top, self.padding.top + height]);

            scatter.selectAll('*')
                   .data(points)
                   .join('g')
                   .append('circle')
                   .attr('r',2)
                   .attr('cx',d=>posXScale(d.x))
                   .attr('cy',d=>posYScale(d.y))
                   .attr("fill",d=>{
                        let rgba = color_scheme[d.cluster]
                        return d3.color(`rgba(${rgba[0] * 255},${rgba[1] *255},${rgba[2] * 255},${rgba[3]})`)
                    })
                    .classed('scatter',true)
                    .classed('highlight_scatter',false)
                    .on('mouseover',(e,d)=>{
                        self.$emit('emitHighlight',d.id)
                    })
                    .on('mouseleave',(e,d)=>{
                        console.log('hh')
                    })
            
        },

        receiveOnHighlight(id){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            this.OffHighlight()
            svg.selectAll('.scatter')
                .each(function(d,i){
                    if(d.id == id){
                        d3.select(this).classed('highlight_scatter',true)
                        d3.select(this).raise()
                    } 
                })
               
        },

        OffHighlight(){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            svg.selectAll('.scatter').classed('highlight_scatter',false)
        }


    },

    mounted() {
        console.log('data:',this.data)
        this.redraw()
    },
}
</script>

<style>
    .vector-embedding-container{
        width: 100%;
        height: 100%;
    }

    .vector-embedding-canvas{
        width:100%;
        height: 100%;
    }

    .highlight_scatter{
        stroke: black;
        stroke-width: 3px;
    }
</style>