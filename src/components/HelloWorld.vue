<template>
  <div>
    <div v-if:="showHeader">
      <h1>hi</h1>
      <h2>This is my prop: {{msg}} </h2>
      <TodoList user="willis"/> 
      <GridTest/>
    </div>
    <ChessBoard/>
  </div>
</template>

<script>
import TodoList from './TodoList.vue'
import ChessBoard from './ChessBoard.vue'
import GridTest from './GridTest.vue'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      showHeader: false
    }
  },
  components: {
    TodoList, ChessBoard, GridTest
  } 
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
