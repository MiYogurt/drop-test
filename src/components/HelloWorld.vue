<template>
  <div class="container" ref="box">
    <div class="padding" ref="padding">
      <div class="hello" :style="style" ref="inner">
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
    return {
      width: 200,
      height: 200
    }
  },
  computed: {
    style(){
      return {
        width: `${this.width}px`,
        height: `${this.height}px`
      }
    }
  },
  methods: {
    paddingSet(left, top){
      this.$refs.padding.style.marginLeft = `${left}px`
      this.$refs.padding.style.marginTop = `${top}px`
    },
    innerSet(x, y){
      this.$refs.inner.style.transform = `translate(-${x}px, -${y}px)`
    }
  },
  mounted(){
    this.$refs.box.addEventListener('mouseup', () => {
      this.canSet = false
      this.startx = 0;
      this.starty = 0;
      console.log('end');
      this.paddingSet(0,0)
      this.$refs.inner.style.transform = `translate(0, 0)`

    }, false)
    this.$refs.box.addEventListener('mousemove', (e) => {
      if(this.canSet){
        const stepx = e.clientX - this.startx ;
        const stepy = e.clientY - this.starty ;
        this.width = this.startStyle.width - stepx;
        this.height = this.startStyle.height - stepy
        this.paddingSet(this.startStyle.width, this.startStyle.height)
        this.innerSet(this.width, this.height)

      }
    }, false)
    this.$refs.box.addEventListener('mousedown', (e) => {
      this.canSet = true
      this.startx = e.clientX;
      this.starty = e.clientY;
      this.startStyle = {
        width: this.width,
        height: this.height
      }
    }, false)
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
.padding{
  padding: 10rem;
  width: max-content;
  height: max-content;
  perspective: 1000;
}
.hello {
  background: red;
  transform-origin: right bottom;
  transform: translateZ(0);
}

</style>
