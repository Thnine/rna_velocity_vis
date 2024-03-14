<template>
  <div id="app">
    <div class="controller-container">
      <el-checkbox v-model="show_arrow" @change="handleShowArrowChange">显示方向</el-checkbox>
    </div>
    <div class="canvas-container">
      <VectorEmbedding ref="vector-embedding" class="vector-embedding" @emitHighlight="handleEmitHighlight" v-for="view in view_list" :key="view['title']" :data="view"></VectorEmbedding>
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
      'show_arrow':false
    }
  },
  methods:{
    handleEmitHighlight(id){
        for(let comp of this.$refs['vector-embedding']){
            comp.receiveOnHighlight(id)
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
    d3.json('static/data.json').then(res=>{
      let data = res;
      let view_list = data['views'].map(v=>{
        let view = JSON.parse(JSON.stringify(v));
        view['color_scheme'] = data['color_scheme'];
        
        // attach id
        for(let i = 0;i < view['points'].length;i++){
          view['points'][i]['id'] = String(i)
        }
        
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
