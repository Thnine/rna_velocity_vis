<template>
  <div id="app">
    <div class="controller-container">
      <!-- <el-checkbox v-model="show_arrow" @change="handleShowArrowChange">显示方向</el-checkbox> -->
      <!-- <el-checkbox v-model="show_arrow" @change="handleShowArrowChange">显示邻居</el-checkbox> -->
      <div>
        <b>邻居数目：</b>
        <el-input-number v-model="neighbor_num" :min="1" label="邻居数目" size="mini"></el-input-number>


        <b>
          邻居类型：
        </b>
        <el-select v-model="neighbor_type" size="mini">
            <el-option
              v-for="item in neighbor_option"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
        </el-select>
      </div>
      
    </div>
    <div class="canvas-container">
      <VectorEmbedding ref="vector-embedding" class="vector-embedding" @emitHighlight="handleEmitHighlight" @emitShowNeighbors="handleEmitShowNeighbors" v-for="view in view_list" :key="view['title']" :data="view"></VectorEmbedding>
    </div>
  </div>
</template>

<script>
import *  as d3 from 'd3'
import VectorEmbedding from './components/VectorEmbedding.vue'

export default {
  name: 'App',
  components: {
    VectorEmbedding
  },
  data(){
    return {
      'view_list':[],
      'show_arrow':false,
      'neighbor_type':'tran',
      'neighbor_num':7,
      'neighbor_option':[{
        'value':'tran',
        'label':'转录'
      },{
        'value':'velo',
        'label':'速率'
      },{
        'value':'comp',
        'label':'复合'
      }]
    }
  },
  methods:{
    handleEmitHighlight(id){
        for(let comp of this.$refs['vector-embedding']){
            comp.receiveOnHighlight(id)
        }
    },
    handleEmitShowNeighbors(id){
        for(let comp of this.$refs['vector-embedding']){
            comp.receiveOnShowNeighbors(id,this.neighbor_type,this.neighbor_num)
        }
    },

    handleShowArrowChange(){
      if(this.show_arrow){
        for(let comp of this.$refs['vector-embedding']){
            comp.showArrow()
        }
      }
      else{
        for(let comp of this.$refs['vector-embedding']){
            comp.hideArrow()
        }
      }
    }
  },
  mounted(){
    d3.json('static/data(test_neighbors2).json').then(res=>{
      let data = res;
      console.log('data:',data)
      let view_list = data['views'].map((v,view_index)=>{
        let view = JSON.parse(JSON.stringify(v));
        view['color_scheme'] = data['color_scheme'];
        // attach id
        for(let i = 0;i < view['points'].length;i++){
          view['points'][i]['id'] = String(i)
        }
        // transfer various neighbors index to id
        let tran_id_neighbors = {}
        let velo_id_neighbors = {}
        let comp_id_neighbors = {}
        for(let i = 0;i < view['points'].length;i++){
          let cur_id = view['points'][i]['id']
          tran_id_neighbors[cur_id] = data['tran_neighbors'][i].map(v=>{
            return view['points'][v]['id']
          })
          velo_id_neighbors[cur_id] = data['velo_neighbors'][i].map(v=>{
            return view['points'][v]['id']
          })
          comp_id_neighbors[cur_id] = data['comp_neighbors'][view_index][i].map(v=>{
            return view['points'][v]['id']
          })
        }
        view['comp_neighbors'] = comp_id_neighbors
        view['velo_neighbors'] = velo_id_neighbors
        view['tran_neighbors'] = tran_id_neighbors
        
        return view
      })
      console.log('all_data:',data)
      this.view_list = view_list
    })
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
}

.canvas-container{
  display: flex;
  flex-wrap: wrap;
}

.vector-embedding{
  flex: 0 0 600px;
  height: 500px;
}
</style>
