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
import {nanoid} from 'nanoid'

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
            },
            dir_length:20,
            arrow_size:4,

            //id
            nanoid:0,

            //其他数据项
            posXScale:null,
            posYScale:null,

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
            const velocity = svg.append('g')

            //title
            svg.append('text')
                .text(title)
                .attr('x',0.5 * width + this.padding.left)
                .attr('y',30)
                .style('font-size',20)

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
            self.posXScale = posXScale
            self.posYScale = posYScale

            //scatter
            scatter.selectAll('*')
                .data(points)
                .join('circle')
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
                    self.$emit('emitShowNeighbors',d.id)
                })
                .on('mouseleave',(e,d)=>{
                })
                .on('click',(e,d)=>{

                })
            


            //velocity
            svg.append('defs')
               .append('marker')
               .attr("id",`velocity_arrow-${self.nanoid}`)
               .attr("markerHeight",self.arrow_size)
               .attr("markerWidth",self.arrow_size)
               .attr("refX",self.arrow_size)
               .attr("refY",0.5 * self.arrow_size)
               .attr("orient","auto")
               .append("path")
               .attr("d",`M 0 0 L ${self.arrow_size} ${0.5 * self.arrow_size} L 0 ${self.arrow_size} z`)

            velocity
                .selectAll('*')
                .data(points)
                .join('line')
                .attr('x1',d=>posXScale(d.x))
                .attr('y1',d=>posYScale(d.y))
                .attr('x2',d=>{
                    let delta_x = posXScale(d.x + d.velocity_x) - posXScale(d.x)
                    let delta_y = posXScale(d.y + d.velocity_y) - posYScale(d.y)

                    if(d.velocity_x != 0 || d.velocity_y != 0)
                        return posXScale(d.x) + delta_x / (Math.sqrt(delta_x * delta_x + delta_y * delta_y)) * self.dir_length
                    else
                        return posXScale(d.x)
                })
                .attr('y2',d=>{
                    let delta_x = posXScale(d.x + d.velocity_x) - posXScale(d.x)
                    let delta_y = posYScale(d.y + d.velocity_y) - posYScale(d.y) 
                    if(d.velocity_x != 0 || d.velocity_y != 0)
                        return posYScale(d.y) + delta_y / (Math.sqrt(delta_x * delta_x + delta_y * delta_y)) * self.dir_length
                    else
                        return posYScale(d.y)
                })
                .style('stroke','black')
                .style('stroke-width','1px')
                .attr("marker-end",(d)=>{
                    if(d.velocity_x != 0 || d.velocity_y != 0)
                        return `url(#velocity_arrow-${self.nanoid})`
                    else
                        return null
                })
                .classed('velocity-arrow',true)
                .style('display','none')
            
            //neighbor links

                
        },

        receiveOnHighlight(id){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])

            svg.selectAll('.scatter').classed('highlight_scatter',false)
            svg.selectAll('.scatter')
                .each(function(d,i){
                    if(d.id == id){
                        d3.select(this).classed('highlight_scatter',true)
                        d3.select(this).raise()
                    } 
                })
        },

        receiveOnShowNeighbors(id){
            this.drawNeighborLinks(id)
        },

        drawNeighborLinks(id){
            /**
             * 画表示neighbor的links
             */ 
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            //clear links
            svg.selectAll('.neighbor-links-plot').remove()
            //prepare pos
            let LUP = {}//id->index的速查表
            for(let i = 0;i < self.data.points.length;i++){
                LUP[self.data.points[i]['id']] = i;
            }
            let base_pos = [self.posXScale(self.data.points[LUP[id]]['x']),self.posYScale(self.data.points[LUP[id]]['y'])]
            let neighbor_pos = self.data.neighbors[LUP[id]].map(v=>{
                return [
                    self.posXScale(self.data.points[LUP[v]]['x']),
                    self.posYScale(self.data.points[LUP[v]]['y'])
                ]
            })
            //draw links
            const neighbor_links_plot = svg.append('g').attr("class","neighbor-links-plot")
            neighbor_links_plot.selectAll('*')
                                .data(neighbor_pos)
                                .join('line')
                                .attr("x1",base_pos[0])
                                .attr("y1",base_pos[1])
                                .attr("x2",(d,i)=>neighbor_pos[i][0])
                                .attr("y2",(d,i)=>neighbor_pos[i][1])
                                .style('stroke','black')
                                .style('stroke-width','0.5px')

        
        },

        showArrow(){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            svg.selectAll('.velocity-arrow')
                .style('display',null)
        },

        hideArrow(){
            const self = this;
            const svg = d3.select(self.$refs['vector-embedding-canvas'])
            svg.selectAll('.velocity-arrow')
                .style('display','none')
        }



    },

    mounted() {
        console.log('data:',this.data)
        this.nanoid = nanoid()
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
        stroke-width: 4px;
    }
</style>