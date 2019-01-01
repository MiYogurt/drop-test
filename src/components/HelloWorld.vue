<template>
  <div class="container" ref="box" @mousemove="movefn">
    <div class="padding" ref="padding" :style="paddingStyle">
      <div class="hello" :style="style" ref="inner"></div>
      <sizer class="abs" :position="position" :mouse="mouse" @change="sync" @start="log('start')" @end="log('end')" @out="log" />
    </div>
  </div>
</template>

<script>
import sizer from './sizer.vue'

export default {
  name: 'HelloWorld',
  components: {
    sizer
  },
  data(){
    return {
      position: {
        left: 1000,
        top: 1000,
        right: 0,
        bottom: 0,
        width: 100,
        height: 100,
      },
      out: false,
      mouse: new MouseEvent('init', {clientX: 0, clientY: 0})
    }
  },
  computed: {
    style(){
      let style = {};
      for (let key of Object.keys(this.position)) {
        if (key == 'right' || key == 'bottom') {
          continue 
        }
        if (key == 'left' || key == 'top') {
          style[key] = this.position[key]  + "px";
          continue 
        }
        style[key] = this.position[key] + "px";
      }
      return style;
    },
    paddingStyle(){
      return {
        'width': Math.abs(this.position.left) + 1000 + 'px',
        'height': Math.abs(this.position.top) + 1000 + 'px'
      }
    }
  },
  methods: {
    outonce(){
      console.log("out");
      
      this.out = true
    },
    log(e){
      console.log(e)
    },
    sync(e){
      console.log(e)
      this.position = e.sync
    },
    movefn(e){
      this.mouse = e
    },
    paddingSet(left, top){
      this.$refs.padding.style.marginLeft = `${left}px`
      this.$refs.padding.style.marginTop = `${top}px`
    },
    innerSet(x, y){
      this.$refs.inner.style.transform = `translate(-${x}px, -${y}px)`
    },
    center(dom){
      console.log(dom);
      
      dom.scrollTo(
        (dom.scrollHeight - dom.clientHeight) / 2,
        (dom.scrollWidth - dom.clientWidth) / 2,
      )
    }
  },
  mounted(){
    this.center(this.$refs.box)
    // this.$refs.box.addEventListener('mouseup', () => {
    //   this.canSet = false
    //   this.startx = 0;
    //   this.starty = 0;
    //   console.log('end');
    //   this.paddingSet(0,0)
    //   this.$refs.inner.style.transform = `translate(0, 0)`

    // }, false)
    // this.$refs.box.addEventListener('mousemove', (e) => {
    //   if(this.canSet){
    //     const stepx = e.clientX - this.startx ;
    //     const stepy = e.clientY - this.starty ;
    //     this.width = this.startStyle.width - stepx;
    //     this.height = this.startStyle.height - stepy
    //     this.paddingSet(this.startStyle.width, this.startStyle.height)
    //     this.innerSet(this.width, this.height)

    //   }
    // }, false)
    // this.$refs.box.addEventListener('mousedown', (e) => {
    //   this.canSet = true
    //   this.startx = e.clientX;
    //   this.starty = e.clientY;
    //   this.startStyle = {
    //     width: this.width,
    //     height: this.height
    //   }
    // }, false)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
*{
  box-sizing: border-box;
}
.container {
  position: relative;
  width: 800px;
  height: 600px;
  background: #f7f7f7;
  overflow: scroll;
}
.container::-webkit-scrollbar {
    display: none;
}
.padding{
  padding: 200px;
  width: max-content;
  height: max-content;
  perspective: 1000;
}
.hello {
  background: red;
  transform-origin: right bottom;
  transform: translateZ(0);
  position: absolute;
}

.fix{
  position: fixed;
}

.abs{
  position: absolute;
}

</style>
