<template>
  <div class="hello"> 
      <button @click="setImage">设置图片</button>
      <croppingImg ref="cropping" :btn="['choose']" @onCropping="cropping" @back="con" />
  </div>
</template>

<script>
import Vue from 'vue'
import toastTest from '../plugin'
import croppingImg from '../plugin/cropping-img/cropping'
Vue.use(toastTest)
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
    }
  },
  components: { croppingImg },
  methods:{
    con() {
      console.log(1)
    },
    cropping(res) {
      console.log(res)
    },
    setImage() {
      console.log(this.$refs.cropping)
      this.$refs.cropping.setImage('http://www.bbra.cn/Uploadfiles/imgs/2017/03/31/feng2/011.jpg')
    }
  }
}
</script>
 
<style>
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
