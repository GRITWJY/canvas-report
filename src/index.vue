<template>
  <div id="canvas-container">
    <div class="container">
      <div class="toolbox">
        <span
          v-for="shape in shapes"
          id="tools"
          :key="shape.id"
          :title="shape.title"
          :class="[
            'canvas-report',
            shape.className,
            { active: config.shape === shape.action },
          ]"
          @click="setShape(shape.action, shape.title)"
        >
          <span style="font-size: 14px; color: orange">{{ shape.key }}</span>
        </span>
      </div>
      <div class="canvasbox">
        <canvas
          id="canvas-Report"
          class="imgcanvas"
          :style="{
            transform:
              'translate(' +
              x +
              'px,' +
              y +
              'px)' +
              'rotate(' +
              deg +
              'deg)scale(' +
              scale +
              ')',
            transformOrigin: leftX + ' ' + topY,
          }"
        ></canvas>
        <canvas
          id="canvas-Correct"
          class="imgcanvas"
          :style="{
            transform:
              'translate(' +
              xa +
              'px,' +
              ya +
              'px)' +
              'rotate(' +
              deg +
              'deg)scale(' +
              scale +
              ')',
            transformOrigin: leftX + ' ' + topY,
          }"
          @mousedown="canvasDown($event)"
          @touchstart="canvasDown($event)"
          @mousemove="canvasMove($event)"
          @touchmove="canvasMove($event)"
          @mouseup="canvasUp($event)"
          @touchend="canvasUp($event)"
        ></canvas>
        <canvas
          id="canvas-Text"
          class="imgcanvas canvasnone"
          :style="{
            transform:
              'translate(' +
              x +
              'px,' +
              y +
              'px)' +
              'rotate(' +
              deg +
              'deg)scale(' +
              scale +
              ')',
          }"
        ></canvas>
        <div id="earse"></div>
        <div class="textbox">
          <el-input
            class="textInput"
            type="textarea"
            :autosize="{ minRows: 1, maxRows: 2 }"
            placeholder="请输入内容"
            v-model="textarea"
          >
          </el-input>
          <el-button
            type="primary"
            plain
            size="mini"
            style="margin: 53px 5px 5px 0; float: right"
            @click="printbox"
            >确定
          </el-button>
          <el-button
            size="mini"
            plain
            style="margin-top: 53px; margin-right: 10px; float: right"
            @click="hidebox"
            >取消
          </el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "canvas-arrow";

export default {
  name: "canvasReport",
  data() {
    return {
      images: [],
      canvasImg: new Image(),
      currentImg: new Image(),
      hanzi: null,
      textarea: "",

      // 批改和报告的画布
      canvasReport: null,
      canvasCorrect: null,
      CanvasSurfaceImageData: null,
      context: null,
      ctx: null,
      imgcanvas: null,
      textInput: null, //文字输入框

      // 鼠标位于画布的距离
      canvasX: null,
      canvasY: null,

      canvasMoveUse: false,
      canvaschanged: false,
      earsebox: null,
      windx: null,
      windy: null,
      config: {
        lineWidth: 6,
        lineColor: "#ff4102",
        shadowBlur: 1,
        lineCap: "round",
        shape: "pencil",
      },
      shapes: [
        {
          className: "icon-pencil tool",
          action: "pencil",
          title: "画笔",
          key: "A",
        },
        {
          className: "icon-eraser tool",
          action: "eraser",
          title: "橡皮擦",
          key: "B",
        },
        {
          className: "icon-arrow tool",
          action: "arrow",
          title: "箭头",
          key: "C",
        },
        {
          className: "icon-rotate-left tool",
          action: "rotateLeft",
          title: "向左旋转",
          key: "D",
        },
        {
          className: "icon-rotate-right tool",
          action: "rotateRight",
          title: "向右旋转",
          key: "E",
        },
        {
          className: "icon-check tool",
          action: "check",
          title: "对勾",
          key: "F",
        },
        {
          className: "icon-cha tool",
          action: "close",
          title: "错误",
          key: "G",
        },
        {
          className: "icon-search-plus tool",
          action: "plus",
          title: "放大",
          key: "H",
        },
        {
          className: "icon-search-minus tool",
          action: "minus",
          title: "缩小",
          key: "I",
        },
        {
          className: "icon-move tool",
          action: "move",
          title: "移动",
          key: "J",
        },
        {
          className: "icon-circle-thin tool",
          action: "circle",
          title: "圆形",
          key: "K",
        },
        {
          className: "icon-square tool",
          action: "square",
          title: "方形",
          key: "L",
        },
        {
          className: "icon-font tool",
          action: "text",
          title: "文字",
          key: "M",
        },
        {
          className: "icon-refresh tool",
          action: "refresh",
          title: "重置或清空",
        },
      ],

      mousedown: {},
      squareUlhc: {},
      circleW: 0,
      circleH: 0,
      // 旋转
      deg: 0,
      rotate_number: 0,
      scale: 1,
      maxscale: 4.9,
      minscale: 0.1,
      y: 0,
      x: 0,
      leftX: "50%",
      topY: "50%",
      xa: 0,
      ya: 0,
      lastX: 0,
      lastY: 0,
      offsetY: 0,
    };
  },

  created() {
    this.init();
    // 按键初始化
    this.initKeyDown();
  },

  methods: {
    /**
     * 按钮绑定事件
     * */
    initKeyDown() {
      document.onkeydown = (e) => {
        switch (e.keyCode) {
          case 65:
            this.setShape("pencil", "画笔");
            break;
          case 66:
            this.setShape("eraser", "橡皮擦");
            break;
          case 67:
            this.setShape("arrow", "箭头");
            break;
          case 68:
            this.setShape("rotateLeft", "向左旋转");
            break;
          case 69:
            this.setShape("rotateRight", "向右旋转");
            break;
          case 70:
            this.setShape("check", "对勾");
            break;
          case 71:
            this.setShape("close", "错误");
            break;
          case 72:
            this.setShape("plus", "放大");
            break;
          case 73:
            this.setShape("minus", "缩小");
            break;
          case 74:
            this.setShape("move", "移动");
            break;
          case 75:
            this.setShape("circle", "圆形");
            break;
          case 76:
            this.setShape("square", "方形");
            break;
          case 77:
            this.setShape("text", "文字");
            break;
        }
      };
    },

    cancelKeyDown() {
      document.onkeydown = () => {};
    },

    // 初始化图片数据
    init() {
      this.canvasImg.src =
        "https://wlsy-hbut.oss-cn-beijing.aliyuncs.com/%E5%AE%9E%E9%AA%8C%E8%B5%84%E6%96%99/empty2.png";
      this.canvasImg.setAttribute("crossOrigin", "Anonymous");
    },
    /**
     画布加载时的情况
     */
    initmouted() {
      // 放着报告原图的canvas
      this.canvasReport = document.querySelector("#canvas-Report");
      // 放着批改痕迹的canvas
      this.canvasCorrect = document.querySelector("#canvas-Correct");

      // 之后用来获取图片宽度的
      this.imgcanvas = document.querySelector(".imgcanvas");

      // 监听画布内的事件
      this.canvasbox = document.querySelector(".canvasbox");

      // 橡皮擦
      this.earsebox = document.querySelector("#earse");

      // 输入框
      this.textInput = document.querySelector(".textbox");

      // 原图片画布的context
      this.context = this.canvasReport.getContext("2d");
      // 批改的context
      this.ctx = this.canvasCorrect.getContext("2d");
      // 这个是用来画文字的
      this.hanzi = this.canvasCorrect.getContext("2d");

      // 画布开始加载
      this.canvasImg.onload = () => {
        // 初始化比例
        let scale = this.reportWidth / this.canvasImg.width;
        // 初始化宽高，这个是由.canvas宽决定的
        let canvasWidth = this.reportWidth;
        let canvasHeight = this.canvasImg.height * scale;

        // 各个图层的宽高都要设定
        this.canvasReport.width = canvasWidth;
        this.canvasReport.height = canvasHeight;
        this.canvasCorrect.width = canvasWidth;
        this.canvasCorrect.height = canvasHeight;
        // 这里是放图片的地方清空，准备放新的图片
        this.context.clearRect(0, 0, canvasWidth, canvasHeight);
        this.context.drawImage(this.canvasImg, 0, 0, canvasWidth, canvasHeight);
        this.changeconfig();
      };
    },

    // 初始化配置， 线的宽度和颜色等
    changeconfig() {
      this.ctx.lineWidth = this.config.lineWidth;
      this.ctx.strokeStyle = this.config.lineColor;
      this.ctx.lineCap = this.config.lineCap;
    },
    // 获取位置
    getcanvasXY(e) {
      this.canvasX = e.offsetX;
      this.canvasY = e.offsetY;
    },
    getwindowXY() {
      let scrollT = this.canvasbox.scrollTop;
      let scrollL = this.canvasbox.scrollLeft;
      let winy = window.event.clientY - this.canvasbox.offsetTop + scrollT;
      let winx = window.event.clientX - this.canvasbox.offsetLeft + scrollL;
      this.windx = winx;
      this.windy = winy;
    },
    getElementLeft(element) {
      return element.offsetLeft;
    },
    getElementTop(element) {
      return element.offsetTop;
    },

    // 画布事件
    canvasDown(e) {
      // 获取坐标
      this.getcanvasXY(e);
      this.mousedown.x = this.canvasX;
      this.disX = e.clientX;
      this.disY = e.clientY;
      this.mousedown.y = this.canvasY;

      // 开始接触画布时，判断是什么事件
      if (this.config.shape === "text") {
        // 如果是文字，因为要输入
        this.cancelKeyDown();
        this.getwindowXY();
        this.textInput.style.display = "block";
        this.textInput.style.left = this.windx + "px";
        this.textInput.style.top = this.windy + "px";
      } else {
        this.canvasMoveUse = true;
        //开始绘制
        this.ctx.beginPath();
        this.ctx.moveTo(this.canvasX, this.canvasY);
        if (this.config.shape == "check") {
          this.canvaschanged = true;
          switch (this.rotate_number) {
            case 1:
              this.ctx.quadraticCurveTo(
                this.canvasX - 80,
                this.canvasY,
                this.canvasX,
                this.canvasY + 100
              );
              break;
            case 2:
              this.ctx.quadraticCurveTo(
                this.canvasX,
                this.canvasY - 80,
                this.canvasX - 100,
                this.canvasY
              );
              break;
            case 3:
              this.ctx.quadraticCurveTo(
                this.canvasX + 80,
                this.canvasY,
                this.canvasX,
                this.canvasY - 100
              );
              break;
            case 0:
              this.ctx.quadraticCurveTo(
                this.canvasX,
                this.canvasY + 80,
                this.canvasX + 100,
                this.canvasY
              );
              break;
          }
          this.ctx.stroke();
        } else if (this.config.shape == "close") {
          this.canvaschanged = true;
          this.ctx.lineTo(this.canvasX + 25, this.canvasY + 25);
          this.ctx.lineTo(this.canvasX - 25, this.canvasY - 25);
          this.ctx.moveTo(this.canvasX, this.canvasY);
          this.ctx.lineTo(this.canvasX + 25, this.canvasY - 25);
          this.ctx.lineTo(this.canvasX - 25, this.canvasY + 25);
          this.ctx.stroke();
        }
        const preData = this.ctx.getImageData(
          0,
          0,
          this.canvasCorrect.width,
          this.canvasCorrect.height
        );
        //在这里把最开始的状态存储下来,为以后话圆和矩形准备
        this.CanvasSurfaceImageData = preData;
      }
    },
    canvasMove(e) {
      if (this.canvasMoveUse) {
        this.getcanvasXY(e);

        if (this.config.shape == "pencil") {
          this.ctx.lineTo(this.canvasX, this.canvasY);
          this.ctx.stroke();
          this.canvaschanged = true;
        } else if (this.config.shape == "eraser") {
          this.earse(this.canvasX, this.canvasY);
          this.canvaschanged = true;
        } else if (this.config.shape == "circle") {
          this.drawcircle();
          this.canvaschanged = true;
        } else if (this.config.shape == "square") {
          this.drawsquare();
          this.canvaschanged = true;
        } else if (this.config.shape == "arrow") {
          this.drawarrow();
          this.canvaschanged = true;
        } else if (this.config.shape == "move") {
          switch (this.rotate_number) {
            case 0:
              this.x = this.canvasX - this.mousedown.x + this.lastX;
              this.y = this.canvasY - this.mousedown.y + this.lastY;
              break;
            case 1:
              console.log(this.offsetY);
              if (this.lastY) {
                this.y = -(this.canvasX - this.mousedown.x) + this.lastY;
              } else {
                this.y =
                  -(this.canvasX - this.mousedown.x) +
                  this.lastY +
                  this.offsetY;
              }

              this.x = this.canvasY - this.mousedown.y + this.lastX;
              break;
            case 2:
              this.x = -(this.canvasX - this.mousedown.x) + this.lastX;
              this.y = -(this.canvasY - this.mousedown.y) + this.lastY;
              break;
            case 3:
              if (this.lastY) {
                this.y = this.canvasX - this.mousedown.x + this.lastY;
              } else {
                this.y =
                  this.canvasX - this.mousedown.x + this.lastY + this.offsetY;
              }
              this.x = -(this.canvasY - this.mousedown.y) + this.lastX;
              break;
          }
        }
      }
    },
    canvasUp(e) {
      this.canvasMoveUse = false;
      this.canvasCorrect.style.display = "block";
      this.ctx.closePath();
      this.xa = this.x;
      this.ya = this.y;
      this.lastX = this.x;
      this.lastY = this.y;
    },

    //选择形状
    setShape(type, title) {
      if (type == "rotateLeft") {
        this.leftrotate();
      } else if (type == "rotateRight") {
        this.rightrotate();
      } else if (type == "refresh") {
        this.canvasbox.onwheel = null;
        this.config.shape = type;
        let isReset = true;
        this.clearing(isReset);
      } else if (type == "plus") {
        if (Number(this.scale).toFixed(1) <= 2.3) {
          this.scale = this.scale + 0.2;
        }
      } else if (type == "minus") {
        if (Number(this.scale).toFixed(1) >= 0.5) {
          this.scale = this.scale - 0.1;
        }
      } else {
        this.canvasbox.onwheel = null;
        this.config.shape = type;
      }
    },

    // 清空,重置
    clearing(isReset) {
      this.canvaschanged = false;
      this.x = 0;
      this.xa = 0;
      this.y = 0;
      this.ya = 0;
      this.deg = 0;
      this.rotate_number = 0;
      this.scale = 1;
      this.leftX = "50%";
      this.topY = "50%";
      this.ctx.clearRect(
        0,
        0,
        this.canvasCorrect.width,
        this.canvasCorrect.height
      );
    },

    // 绘制特殊形状
    drawarrow() {
      this.ctx.fillStyle = this.config.lineColor;
      this.ctx.putImageData(this.CanvasSurfaceImageData, 0, 0);
      this.ctx.save();
      this.ctx.beginPath();
      this.ctx.drawArrow(
        this.mousedown.x,
        this.mousedown.y,
        this.canvasX,
        this.canvasY,
        this.config.lineWidth
      );
      this.ctx.fill();
      this.ctx.restore();
    },

    // 橡皮擦
    earse(x, y) {
      let earse = document.querySelector("#earse");
      this.earsebox.style.display = "block";
      this.getwindowXY();
      this.windy -=
        document.querySelector("#canvas-container").offsetParent.offsetTop || 0;
      document.onmousemove = () => {
        if (this.rotate_number == 1) {
          this.earsebox.style.left = this.windx + "px";
          this.earsebox.style.top = this.windy - 60 * this.scale + "px";
        } else if (this.rotate_number == 2) {
          this.earsebox.style.left = this.windx - 60 * this.scale + "px";
          this.earsebox.style.top = this.windy - 60 * this.scale + "px";
        } else if (this.rotate_number == 3) {
          this.earsebox.style.left = this.windx - 60 * this.scale + "px";
          this.earsebox.style.top = this.windy + "px";
        } else if (this.rotate_number == 0) {
          this.earsebox.style.left = this.windx + "px";
          this.earsebox.style.top = this.windy + "px";
        }

        this.earsebox.style.width = 60 * this.scale + "px";
        this.earsebox.style.height = 60 * this.scale + "px";
      };
      document.onmouseup = () => {
        this.canvasMoveUse = false;
        this.earsebox.style.display = "none";
      };
      this.ctx.clearRect(this.canvasX, this.canvasY, 60, 60);
    },

    // 隐藏文字的box
    hidebox() {
      this.initKeyDown();
      this.textInput.style.display = "none";
    },
    //  绘制文字
    printbox() {
      this.initKeyDown();
      this.canvaschanged = true;
      this.textInput.style.display = "none";
      this.hanzi.font = this.config.lineWidth * 6 + "px" + " Arial";
      this.hanzi.fillStyle = this.config.lineColor;
      if (this.rotate_number == 1) {
        this.hanzi.rotate(Math.PI / 2);
        this.hanzi.fillText(this.textarea, this.canvasY, -this.canvasX);
        this.hanzi.rotate(-Math.PI / 2);
      } else if (this.rotate_number == 2) {
        this.hanzi.rotate(Math.PI);
        this.hanzi.fillText(this.textarea, -this.canvasX, -this.canvasY);
        this.hanzi.rotate(-Math.PI);
      } else if (this.rotate_number == 3) {
        this.hanzi.rotate((Math.PI * 3) / 2);
        this.hanzi.fillText(this.textarea, -this.canvasY, this.canvasX);
        this.hanzi.rotate((-Math.PI * 3) / 2);
      } else {
        this.hanzi.fillText(this.textarea, this.canvasX, this.canvasY);
      }
    },

    // 绘制圆
    updateCircleRadius(x, y) {
      this.circleW = x - this.mousedown.x;
      this.circleH = y - this.mousedown.y;

      if (x > this.mousedown.x) {
        this.squareUlhc.x = this.mousedown.x;
      } else {
        this.squareUlhc.x = x;
      }

      if (y > this.mousedown.y) {
        this.squareUlhc.y = this.mousedown.y;
      } else {
        this.squareUlhc.y = y;
      }
    },
    drawcircle() {
      this.ctx.putImageData(this.CanvasSurfaceImageData, 0, 0);
      this.updateCircleRadius(this.canvasX, this.canvasY);
      this.ctx.save();
      this.ctx.beginPath();
      this.ctx.ellipse(
        this.canvasX - this.circleW / 2,
        this.canvasY - this.circleH / 2,
        Math.abs(this.circleW / 2),
        Math.abs(this.circleH / 2),
        0,
        0,
        Math.PI * 2
      );
      this.ctx.stroke();
      this.ctx.restore();
    },
    //矩形
    drawsquare() {
      this.ctx.putImageData(this.CanvasSurfaceImageData, 0, 0);
      this.updateCircleRadius(this.canvasX, this.canvasY);
      this.ctx.strokeRect(
        this.squareUlhc.x,
        this.squareUlhc.y,
        Math.abs(this.circleW),
        Math.abs(this.circleH)
      );
      this.ctx.restore();
    },

    // 左旋转
    leftrotate() {
      this.deg -= 90;
      this.rotate_number += 1;
      if (this.rotate_number == 4) {
        this.rotate_number = 0;
      }
      let degree = 90 / 360;

      if (this.rotate_number % 2) {
        console.log(this.canvasReport);
        let chaju = this.canvasReport.width - this.canvasReport.height;
        if (chaju > 0) {
          //宽比高长
          this.y = Math.abs(chaju);
          this.ya = Math.abs(chaju);
          this.offsetY = Math.abs(chaju);
        } else {
          this.y = chaju / 2;
          this.ya = chaju / 2;
          this.offsetY = chaju / 2;
        }
        this.scale = this.canvasReport.width / this.canvasReport.height;
      } else {
        this.scale = 1;
        this.y = 0;
        this.ya = 0;
      }

      this.ctx.translate(
        this.canvasCorrect.width / 2,
        this.canvasCorrect.height / 2
      );
      this.context.translate(
        this.canvasReport.width / 2,
        this.canvasReport.height / 2
      );
      this.ctx.rotate((degree / 100) * Math.PI);
      this.context.rotate((degree / 100) * Math.PI);
      this.ctx.translate(
        -this.canvasCorrect.width / 2,
        -this.canvasCorrect.height / 2
      );
      this.context.translate(
        -this.canvasReport.width / 2,
        -this.canvasReport.height / 2
      );
    },
    rightrotate() {
      this.deg += 90;
      this.rotate_number -= 1;
      if (this.rotate_number == -1) {
        this.rotate_number = 3;
      }
      let degree = -90 / 360;

      if (this.rotate_number % 2) {
        this.y =
          -Math.abs(this.canvasReport.height - this.canvasReport.width) / 2;
        this.ya =
          -Math.abs(this.canvasReport.height - this.canvasReport.width) / 2;
        this.offsetY =
          -Math.abs(this.canvasReport.height - this.canvasReport.width) / 2;
        this.scale = this.canvasReport.width / this.canvasReport.height;
      } else {
        this.y = 0;
        this.ya = 0;
        this.scale = 1;
      }

      this.ctx.translate(
        this.canvasCorrect.width / 2,
        this.canvasCorrect.height / 2
      );
      this.context.translate(
        this.canvasReport.width / 2,
        this.canvasReport.height / 2
      );

      this.ctx.rotate((degree / 100) * Math.PI);
      this.context.rotate((degree / 100) * Math.PI);

      this.ctx.translate(
        -this.canvasCorrect.width / 2,
        -this.canvasCorrect.height / 2
      );
      this.context.translate(
        -this.canvasReport.width / 2,
        -this.canvasReport.height / 2
      );
    },
  },
  computed: {
    reportWidth() {
      return this.imgcanvas.clientWidth;
    },
  },

  mounted() {
    this.initmouted();
  },
};
</script>

<style scoped>
@import "./styles/index.css";
</style>

<style scoped lang="scss">
.inline-input {
  display: block;
  width: 100%;
}

.selected {
  border: 2px solid red;
}

.icon-click:hover {
  color: #037beb;
  cursor: pointer;
}

.icon-click {
  color: #94c0f9;
  font-size: 24px;
}

.scorebtn {
  margin: 8px 5px;
  width: 2.5rem;
  height: 1.5rem;
  padding: 1px;
}

.fonteditor {
  width: 100px;
  height: 40px;
  margin: 5px 0 5px 40px;
  border-radius: 5px;
  display: flex;
  flex-direction: row;

  input {
    height: 20px;
    margin: auto;
    width: 50px;
    border-radius: 6px;
  }
}

.topbox {
  background-color: rgba(58, 67, 87, 0.7);
  width: 100vw;
  height: 50px;
  position: fixed;
}

#earse {
  display: none;
  width: 60px;
  height: 60px;
  background-color: white;
  position: absolute;
}

#preport1 {
  width: 70px;
  height: 100px;
  overflow: hidden;
}

#preport2 {
  width: 70px;
  height: 100px;
  overflow: hidden;
}

#preport3 {
  width: 70px;
  height: 100px;
  overflow: hidden;
}

.container {
  display: flex;
}

.imgcanvas {
  background-color: rgba(0, 0, 0, 0);
  float: right;
  position: absolute;
  width: 100%;
  min-width: 750px;
}

.canvasbox {
  position: relative;
  width: 60vw;
  min-width: 750px;
  max-width: 800px;
  overflow-x: hidden;
  overflow-y: scroll;
}

.reportinfo {
  min-width: 450px;
  max-width: 450px;
  border-radius: 1rem;
  height: calc(100vh - 120px);
  background-color: #f2f4f7;
}

.stuavatar {
  width: 5rem;
  height: 5rem;
  margin: 1rem 1rem 1rem 1.5rem;
  float: left;
}

.stuindex {
  float: right;
  margin: 1rem 1rem auto;
  font-weight: bold;
  font-size: 1.1rem;
  color: #8d9196;
}

.stuinfo {
  float: left;
  display: flex;
  flex-direction: column;
  margin-top: 1rem;
}

.stuinfo span {
  padding-bottom: 5px;
}

.toolbox {
  width: 50px;
  display: flex;
  background-color: #f2f4f7;
  padding-bottom: 20px;
  border-radius: 20px;
  flex-direction: column;
  text-align: center;
  height: 664px;
  min-width: 50px;
}

.tool {
  margin-top: 20px;
  font-size: 24px;
  color: #afc3dd;
}

.active {
  color: white;
  border: 1px solid;
  background-color: #94c0f9;
  width: 40px;
  margin: 25px auto 0 auto;
  height: 35px;
  border-radius: 5px;
  padding-top: 5px;
}

.tool:hover {
  cursor: pointer;
}

.textInput {
  position: absolute;
  width: 300px;
  height: 60px;
}

.textbox {
  height: 90px;
  display: none;
  position: absolute;
  width: 300px;
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 5px;
}

.canvasnone {
  display: none;
}
</style>
