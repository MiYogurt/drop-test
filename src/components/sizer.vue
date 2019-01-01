<template>
  <div
    class="sizer"
    :class="{grabbing: active}"
    @dragstart="noop"
    :style="containerStyle"
    @mousedown.self="shouldActive('move', $event)"
    @mouseup="forbidenActive"
  >
    <div
      class="top"
      :class="{ hover: topHover }"
      @mousedown="shouldActive('t', $event)"
    ></div>
    <div
      class="right"
      :class="{ hover: rightHover }"
      @mousedown="shouldActive('r', $event)"
    ></div>
    <div
      class="bottom"
      :class="{ hover: bottomHover }"
      @mousedown="shouldActive('b', $event)"
    ></div>
    <div
      class="left"
      :class="{ hover: leftHover }"
      @mousedown="shouldActive('l', $event)"
    ></div>
    <!-- left top -->
    <div
      class="lt"
      @mouseenter="mouseenter('lt')"
      @mousedown="shouldActive('lt', $event)"
      @mouseout="leave"
    ></div>
    <!-- right top -->
    <div
      class="rt"
      @mouseenter="mouseenter('rt')"
      @mousedown="shouldActive('rt', $event)"
      @mouseout="leave"
    ></div>
    <!-- left bottom -->
    <div
      class="lb"
      @mouseenter="mouseenter('lb')"
      @mousedown="shouldActive('lb', $event)"
      @mouseout="leave"
    ></div>
    <!-- right bottom -->
    <div
      class="rb"
      @mouseenter="mouseenter('rb')"
      @mousedown="shouldActive('rb', $event)"
      @mouseout="leave"
    ></div>
  </div>
</template>
<script>
export default {
  name: "Sizer",
  props: {
    position: {
      default() {
        return {
          left: 0,
          top: 0,
          right: 0,
          bottom: 0,
          width: 100,
          height: 100
        };
      }
    },
    mouse: {
      type: MouseEvent,
      required: true
    },
    lock: {
      default: () => ({
        left: false,
        top: false,
        width: false,
        height: false,
        right: true,
        bottom: true
      })
		},
		detection: {
			default: 1000
		}
  },
  data() {
    return {
      innner_position: this.position,
      hover: {
        lt: false,
        rt: false,
        lb: false,
        rb: false
      },
      type: "none",
      active: false,
      distance: {
        x: 0,
        y: 0
      }
    };
  },
  computed: {
    containerStyle() {
      let style = {};
      for (let key of Object.keys(this.innner_position || this.position)) {
        style[key] = this.position[key] + "px";
      }
      return style;
    },
    topHover() {
      return this.hover.lt || this.hover.rt;
    },
    leftHover() {
      return this.hover.lt || this.hover.lb;
    },
    rightHover() {
      return this.hover.rb || this.hover.rt;
    },
    bottomHover() {
      return this.hover.lb || this.hover.rb;
    }
  },
  methods: {
    noop(e) {
      e.preventDefault();
    },
    shouldActive(type, e) {
      this.type = type;
      this.active = true;
      this.cache = this.getPoint(e);
			this.start = Object.assign({}, this.innner_position);
			this.$emit("start", this.type)
    },
    getPoint(e) {
      return {
        x: e.clientX,
        y: e.clientY
      };
    },
    getDistance(e) {
      this.distance = this.subPoint(this.getPoint(e), this.cache);
      return this.distance;
    },
    subPoint(p1, p2) {
      return {
        x: p1.x - p2.x,
        y: p1.y - p2.y
      };
    },
    setLeft() {
      if (this.lock.left) {
        return;
      }
      this.innner_position.left = this.start.left + this.distance.x;
    },
    setRight() {
      if (this.lock.right) {
        return;
      }
      this.innner_position.right = this.start.right + this.distance.x;
    },
    setTop() {
      if (this.lock.top) {
        return;
      }
      this.innner_position.top = this.start.top + this.distance.y;
    },
    setBottom() {
      if (this.lock.bottom) {
        return;
      }
      this.innner_position.bottom = this.start.bottom + this.distance.y;
    },
    setHeight(revert) {
      if (this.lock.height) {
        return;
      }
      if (revert) {
        this.innner_position.height = this.start.height - this.distance.y;
        return;
      }
      this.innner_position.height = this.start.height + this.distance.y;
    },
    setWidth(revert) {
      if (this.lock.width) {
        return;
      }
      if (revert) {
        this.innner_position.width = this.start.width - this.distance.x;
        return;
      }
      this.innner_position.width = this.start.width + this.distance.x;
    },
    forbidenActive() {
      this.type = "none";
			this.active = false;
			this.$emit("end", this.type)
		},
    upload() {
      this.$emit("change", {
				type: this.type,
				start: this.start,
        distance: {
					x: this.distance.x,
					y: this.distance.y
				},
				sync: this.innner_position
      });
    },
    mousemove(e) {
      if (!this.active) {
        return;
      }
			const self = this;
			
      const fns = {
        move() {
          self.setLeft();
          self.setTop();
        },
        l() {
          self.setLeft();
          self.setWidth(true);
        },
        r() {
          self.setRight();
          self.setWidth();
        },
        t() {
          self.setTop();
          self.setHeight(true);
        },
        b() {
          self.setBottom();
          self.setHeight();
        },
        lt() {
          this.l();
          this.t();
        },
        rt() {
          this.r();
          this.t();
        },
        lb() {
          this.l();
          this.b();
        },
        rb() {
          this.r();
          this.b();
        }
      };

      this.getDistance(e);
			fns[this.type]()
      this.upload();
    },
    mouseenter(position) {
      this.hover[position] = true;
    },
    leave() {
      this.hover = {
        lt: false,
        rt: false,
        lb: false,
        rb: false
      };
		},
		throttle(name, fn){
			if (this['timer' + name]) {
				return
			}
			this['timer' + name] = setTimeout(() => {
				fn()
				clearTimeout(this['timer' + name])
				this['timer' + name] = 0
			}, this.detection);
		},
		handleOut(e){
			this.throttle('checkInner', () => {
				if(this.$el.contains(e.target) || this.$el === e.target){
						return
				} 
				this.active = false
					this.$emit("out")
			})
		}
  },
  watch: {
    mouse(e) {
			this.mousemove(e);
			if(this.active && this.detection != 0){
				this.handleOut(e)
			}
    }
  }
};
</script>
<style lang="stylus" scoped>
.sizer{
    background rgba(255,255,255, .7)
    box-shadow inset 1px 1px 20px 0px rgba(8, 115, 222, 0.7)
    position relative
    cursor grab
}

.grabbing{
    cursor grabbing
}

.top,
.bottom,
.right ,
.left {
    position absolute
    color #86b3ff
    &:hover,
    &.hover{
        border-color #0773de
    }
}

.lt,
.rt,
.lb,
.rb{
    position absolute
    width 12px
    height 12px
    background #0773de
    border-radius 50%
    &:hover {
        background #61e6b8
    }
}

.lt{
    left -3px
    top -3px
    cursor nw-resize
}

.rt {
    right -3px
    top -3px
    cursor ne-resize
}

.lb{
    bottom -3px
    left -3px
    cursor sw-resize
}

.rb{
    right -3px
    bottom -3px
    cursor se-resize
}

.top,
.bottom {
    height 5px
    left 0
    right 0
    cursor ns-resize
}

.top {
    top 0
    border-top: 4px solid
}

.bottom {
    bottom 0
    border-bottom: 4px solid
}

.left,
.right {
    width 5px
    top 0
    bottom 0
    cursor ew-resize
}

.left {
    left 0
    border-left: 4px solid
}

.right {
    right 0
    border-right: 4px solid
}
</style>