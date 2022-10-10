<template>
  <div>
    <canvas
      @mousedown="down($event)"
      @mousemove="move($event)"
      @mouseup="up"
      @mouseenter="enter"
      @touchstart="start($event)"
      @touchmove="move2($event)"
      @touchend="up"
      id="canvas"
      class="cursor1"
      width="1300"
      height="900"
    ></canvas>
    <div id="actions" class="actions">
      <el-button @click="beginPencil" type="primary">画笔</el-button>
      <el-button @click="beginEraser" type="primary">橡皮擦</el-button>
      <el-button @click="save">下载</el-button>
      <el-button @click="clear" type="danger">清屏</el-button>
      <el-button :disabled="isdisabled" type="primary" @click="cancel">撤回</el-button>
    </div>
    <ul class="colors">
      <li style="width: 200px; margin-top: 50px">画笔颜色选择</li>
      <li @click="black" id="black" class="black"></li>
      <li @click="red" id="red" class="red"></li>
      <li @click="orange" id="orange" class="orange"></li>
      <li @click="green" id="green" class="green"></li>
      <li @click="blue" id="blue" class="blue"></li>
      <li><el-color-picker @change="selectChange" v-model="color1"></el-color-picker></li>
      <li style="width: 200px; margin-top: 50px">画笔粗细选择</li>
      <li @click="line1" class="line1"></li>
      <li @click="line2" class="line2"></li>
      <li @click="line3" class="line3"></li>
    </ul>
  </div>
</template>


<script>
export default {
  name: 'App',
  data() {
    return {
      canvas: null,
      ctx: null,
      startPoint: {
        x: 0,
        y: 0
      },
      endPoint: {
        x: 0,
        y: 0
      }, //坐标
      painting: true, //画板控制开关
      eraser: false, //橡皮擦
      history: [], //存储当前表面状态数组-上一步
      isdisabled: false,
      color1: '#000000'
    }
  },
  mounted() {
    this.initCanvas()
    this.isdisabled = true
  },
  methods: {
    selectChange(value) {
      // console.log(value)
      this.ctx.strokeStyle = value
    },
    line1() {
      this.ctx.lineWidth = 3
    },
    line2() {
      this.ctx.lineWidth = 15
    },
    line3() {
      this.ctx.lineWidth = 25
    },
    //黑画笔
    black() {
      this.ctx.strokeStyle = 'black'
      canvas.classList.add('cursor1')
      canvas.classList.remove('cursor2')
      canvas.classList.remove('cursor3')
      canvas.classList.remove('cursor4')
      canvas.classList.remove('cursor5')
      canvas.classList.remove('xiangpica')
      this.eraser = false
    },
    //红画笔
    red() {
      this.ctx.strokeStyle = 'red'
      canvas.classList.add('cursor2')
      canvas.classList.remove('cursor1')
      canvas.classList.remove('cursor3')
      canvas.classList.remove('cursor4')
      canvas.classList.remove('cursor5')
      canvas.classList.remove('xiangpica')
      this.eraser = false
    },
    orange() {
      this.ctx.strokeStyle = 'orange'
      canvas.classList.add('cursor3')
      canvas.classList.remove('cursor1')
      canvas.classList.remove('cursor2')
      canvas.classList.remove('cursor4')
      canvas.classList.remove('cursor5')
      canvas.classList.remove('xiangpica')
      this.eraser = false
    },
    green() {
      this.ctx.strokeStyle = 'green'
      canvas.classList.add('cursor4')
      canvas.classList.remove('cursor1')
      canvas.classList.remove('cursor2')
      canvas.classList.remove('cursor3')
      canvas.classList.remove('cursor5')
      canvas.classList.remove('xiangpica')
      this.eraser = false
    },
    blue() {
      this.ctx.strokeStyle = 'blue'
      canvas.classList.add('cursor5')
      canvas.classList.remove('cursor1')
      canvas.classList.remove('cursor2')
      canvas.classList.remove('cursor3')
      canvas.classList.remove('cursor4')
      canvas.classList.remove('xiangpica')
      this.eraser = false
    },
    beginEraser() {
      this.eraser = true
      canvas.classList.add('xiangpica')
    },
    beginPencil() {
      this.eraser = false
      canvas.classList.remove('xiangpica')
    },
    initCanvas() {
      this.canvas = document.getElementById('canvas')
      // this.canvas.width = width+'px'
      // this.canvas.height = height+'px'
      this.ctx = canvas.getContext('2d')
    },
    down(e) {
      // 鼠标按下事件
      const x = e.offsetX
      const y = e.offsetY

      this.startPoint = {
        x: x,
        y: y
      }

      if (this.eraser) {
        // 启用橡皮擦
        this.ctx.clearRect(x, y, 20, 20)
      }

      this.painting = true
    },

    move(e) {
      // 鼠标移动事件
      e.preventDefault()
      const x = e.offsetX
      const y = e.offsetY

      this.endPoint = {
        x: x,
        y: y
      }

      if (this.painting) {
        if (this.eraser) {
          this.ctx.clearRect(x, y, 20, 20)
        } else {
          this.draw()
        }
      }

      this.startPoint.x = this.endPoint.x
      this.startPoint.y = this.endPoint.y
    },

    up() {
      // 鼠标松开事件

      // 绘画完成，推入历史记录
      let image = this.ctx.getImageData(0, 0, 1300, 900)
      this.history.push(image)
      if (this.history.length >= 1) {
        this.isdisabled = false
      }
      // history中有记录，撤回按钮可用
      // bus.$emit('abled', '启用撤回')

      this.painting = false
    },
    enter() {
      // 修复pc端使用橡皮擦之后进入canvas立即开始画线
      this.painting = false
    },
    cancel() {
      // 撤回上一步
      this.history.pop()

      if (this.history.length == 0) {
        this.clear()
        this.isdisabled = true
        // alert("无法撤回，请清空！")
        // bus.$emit('disabled', '禁用撤回')
      } else {
        this.ctx.putImageData(this.history[this.history.length - 1], 0, 0)
      }
    },

    draw() {
      // 开始绘制
      this.ctx.beginPath()
      // 设置线条样式
      this.ctx.lineWidth = this.value
      this.ctx.lineCap = 'round'
      this.ctx.lineJoin = 'round'
      this.ctx.strokeStyle = this.color
      //起始位置
      this.ctx.moveTo(this.startPoint.x, this.startPoint.y)
      //停止位置
      this.ctx.lineTo(this.endPoint.x, this.endPoint.y)
      //描绘线路
      this.ctx.stroke()
      //结束绘制
      this.ctx.closePath()
    },
    //清屏
    clear() {
      this.ctx.fillStyle = '#C5C5C5'
      this.ctx.fillRect(0, 0, canvas.width, canvas.height)
      this.history = []
      this.isdisabled = true
    },
    save() {
      let url = canvas.toDataURL('image/jpg')
      let a = document.createElement('a')
      document.body.appendChild(a)
      a.href = url
      a.download = '草稿纸'
      a.target = '_blank'
      a.click()
    },

    start(e) {
      //[0]表示touch第一个触碰点
      let x = e.touches[0].clientX
      let y = e.touches[0].clientY

      this.startPoint = {
        x: x,
        y: y
      }

      if (this.eraser) {
        this.ctx.clearRect(x, y, 10, 10)
      }

      this.painting = true
    },

    move2(e) {
      e.preventDefault()
      let x = e.touches[0].clientX
      let y = e.touches[0].clientY
      this.endPoint = {
        x: x,
        y: y
      }

      if (this.painting) {
        if (this.eraser) {
          this.cxt.clearRect(x, y, 10, 10)
        } else {
          this.draw()
        }
      }

      this.startPoint.x = this.endPoint.x
      this.startPoint.y = this.endPoint.y
    }
  }
}
</script>

<style lang="less" scoped>
.icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
* {
  margin: 0;
  padding: 0;
}
ul,
ol {
  list-style: none;
}
#canvas {
  position: fixed;
  top: 0;
  left: 0;
  background: #c5c5c5;
  display: block;
  overflow: hidden;
}
.actions {
  position: fixed;
  top: 0;
  left: 0;
}
.actions > svg {
  width: 1.5rem;
  height: 1.5rem;
  margin: 0.5rem 1rem;
  transition: all 0.3s;
}
.actions svg.active {
  fill: orangered;
  transform: scale(1.4);
}
.colors {
  position: fixed;
  top: 4rem;
  left: 0.5rem;
}
.colors > li {
  margin: 1rem 0;
  width: 2rem;
  height: 2rem;
  // border-radius: 1rem;
  // box-shadow: 0px 0px 3px black;
}
.black {
  background: black;
}
.red {
  background: red;
}
.orange {
  background: orange;
}
.green {
  background: green;
}
.blue {
  background: blueviolet;
}
.cursor1 {
  cursor: url('assets/ico/black.ico') 8 20, auto;
}
.cursor2 {
  cursor: url('assets/ico/red.ico') 8 20, auto;
}
.cursor3 {
  cursor: url('assets/ico/orange.ico') 8 20, auto;
}
.cursor4 {
  cursor: url('assets/ico/green.ico') 8 20, auto;
}
.cursor5 {
  cursor: url('assets/ico/blue.ico') 8 20, auto;
}
.xiangpica {
  cursor: url('assets/ico/xiangpica.ico') 8 20, auto;
}
.line1 {
  width: 100px !important;
  height: 5px !important;
  background: #000;
  cursor: pointer;
}
.line2 {
  width: 100px !important;
  height: 15px !important;
  background: #000;
  cursor: pointer;
}
.line3 {
  width: 100px !important;
  height: 20px !important;
  background: #000;
  cursor: pointer;
}
</style>
