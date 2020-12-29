<template>
  <div class="hello">
      <vedit :container="container" :height="1000" :width="1820"></vedit>
  </div>
</template>

<script>
import Vedit from './vedit'
import Draw from './draw'
// import Test from './test'
export default {
  name: 'HelloWorld',
  components: {
    Vedit, 
    Draw,
    // Test
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      leftShow: true,
      rightShow: true,
      container: 'middle'
    }
  },
  methods: {
    foldBox (p) {
      p === 'right' ? (this.rightShow = !this.rightShow) : (this.leftShow = !this.leftShow)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
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
.hello{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
}
.left{
  width: 200px;
  height: 100%;
  box-shadow: 0 0 1px blue;
}
.middle{
  flex: 1;
  box-shadow: 0 0 1px red;
  background: #666;
}
.right{
  width: 200px;
  height: 100%;
  box-shadow: 0 0 1px green;
}
.foldButton{
  width: 20px;
  height: 30px;
  background: pink;
  cursor: pointer;
}
.b_right{
  float: right;
}
.b_left{
  float: left;
}
</style>
