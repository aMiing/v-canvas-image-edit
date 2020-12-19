<template>
  <div id="dialog">
    <div class="toolBar">
      <fieldset style="float: left">
        <legend>工具</legend>
        <div class="toolsOptions">
          <span v-for="item in toolBar.tool" :key="item.name">
            <input
              type="radio"
              name="toolsOption"
              :id="'tools_' + item.name"
              @change="changeTool(item.name)"
              :checked="item.name == currentTool"
            />
            <label :for="'tools_' + item.name">{{ item.display }}</label>
          </span>

          <span v-for="item in toolBar.shade" :key="item.name">
            <input
              type="radio"
              name="toolsOption"
              :id="'tools_' + item.name"
              @change="changeTool(item.name)"
              :checked="item.name == currentTool"
            />
            <label :for="'tools_' + item.name" :style="'color:red;background: url('+item.icon+')'">{{ item.display }}</label>
          </span>

        </div>
      </fieldset>
      <fieldset style="float: left; margin-left: 20px">
        <legend>操作</legend>
        <div class="toolsOptions">
            <label 
                v-for="item in toolBar.opration"
                :key="item.name"
                :id="'tools_' + item.name"
                @click="oprationClickHandle(item.name)">
                {{ item.display }}
            </label>
            <span
            >边框色:
            <el-color-picker
                v-model="borderColor"
                size="mini"
                @change="borderColorChange"
            ></el-color-picker>
            </span>
            <span
            >文字颜色:
            <el-color-picker
                v-model="textColor"
                size="mini"
                @change="textColorChange"
            ></el-color-picker>
            </span>
        </div>
      </fieldset>

      <fieldset style="float: left; margin-left: 20px">
        <legend>文件操作</legend>
        <div class="toolsOptions">
            <label id="tools_uploadImg" @click="uploadImg">上传图片</label>
        </div>
      </fieldset>

      <div style="clear: both"></div>
    </div>

    <div
      style="background: white; position: relative; margin-top: 5px"
      id="container"
      @mousemove="mouseMoveEventHandler"
      @mousedown="mouseDownEventHandler"
      @mouseup="mouseupEventHandler"
    >
      <div
        id="temp"
        style="
          border: 1px solid gray;
          width: 1px;
          height: 1px;
          position: absolute;
          display: none;
        "
      ></div>

      <canvas
        ref="myCanvas"
        id="myCanvas"
        :width="canvasSize.w"
        :height="canvasSize.h"
        :class="containerClass"
        style="position: absolute; top: 1px; left: 1px; background: #fff"
      >
      </canvas>
      <canvas
        ref="myCanvas1"
        id="myCanvas1"
        :width="canvasSize.w"
        :height="canvasSize.h"
        class="container_pencil"
        style="position: absolute; top: 1px; left: 1px"
        @mousemove="canvas1Mousemove"
        @mouseup="canvas1Mouseup"
      >
      </canvas>
      <div
        class="anyLine"
        v-show="anyLineShow"
        :style="
          'border-top: 1px solid ' +
          borderColor +
          ';width: ' +
          lineLength +
          'px;transform: ' +
          lineTransform +
          ';left: ' +
          lineLeft +
          'px;top: ' +
          lineTop +
          'px'
        "
      ></div>
      <div
        class="circleDom"
        ref="circleDom"
        v-show="rectTipShow"
        :style="
          'border: 1px solid ' +
          borderColor +
          ';width: 1px;height: 1px;position: absolute;'
        "
      ></div>
      <textarea
        ref="fontTip"
        v-show="fontTipObj.show"
        class="fontTip"
        rows="3"
        cols="20"
        @blur="drawWords"
        v-model="textValue"
        :style="
          'top:' +
          fontTipObj.top +
          'px;left:' +
          fontTipObj.left +
          'px;width:' +
          fontTipObj.width +
          'px;height:' +
          fontTipObj.height +
          'px;color: ' +
          fontTipObj.textColor
        "
      ></textarea>
    </div>
  </div>
</template>

<script>
const toolbar = {
  tool: [
    {
      name: "pencil",
      display: "画笔",
      icon: 'images/pencil.svg',
    },
    {
      name: "eraser",
      display: "橡皮擦",
      icon: 'images/eraser.svg',
    },
    {
      name: "trash",
      display: "清空",
      icon: 'images/clear.svg',
    },
  ],
  opration: [
    {
      name: "save",
      display: "save",
      icon: 'images/save.svg',
    },
    {
      name: "undo",
      display: "undo",
      icon: 'images/undo.svg',
    },
    {
      name: "redo",
      display: "redo",
      icon: 'images/redo.svg',
    },
  ],
  shade: [
    {
      name: "line",
      display: "直线",
      icon: 'images/line.svg',
    },
    {
      name: "circle",
      display: "圆形",
      icon: 'images/circle.svg',
    },
    {
      name: "rect",
      display: "矩形",
      icon: 'images/rect.svg',
    },
    {
      name: "text",
      display: "文字",
      icon: 'images/text.svg',
    },
  ],
};

// Object.freeze(toolbar)
let width = window.innerWidth;
let height = window.innerHeight;
window.onresize = function(){
     width = window.innerWidth;
    height = window.innerHeight;
}
export default {
  name: "edit",
  props: {
      width: {
          type: Number,
          default: 0
      },
      height: {
          type: Number,
          default: 0
      },
  },
  data() {
    return {
      flag: false,
      anyLineShow: false,
      rectTipShow: false,
      textValue: "",
      fontTipObj: {
        show: false,
        top: "0px",
        left: "0px",
        width: "0px",
        height: "0px",
      },
      canvasSize: {
        w: (this.width || width) - 404,
        h: (this.height || height) - 120,
      },
      containerClass: "container_pencil",
      toolBar: toolbar,
      currentTool: "pencil",
      cStep: -1, //记录步伐，用于回退前进
      history: [],
      borderColor: "#333",
      textColor: "#000",
      lineLength: 1,
      lineTransform: "0deg",
      lineLeft: 0,
      lineTop: 0,
    };
  },
  mounted() {
    this.$nextTick(() => {
      this.canvas = this.$refs.myCanvas;
      this.canvas1 = this.$refs.myCanvas1;
      this.ctx = this.canvas.getContext("2d");
      this.ctx1 = this.canvas1.getContext("2d");

      this.canvasLeft = this.canvas.offsetParent.offsetLeft;
      this.canvasTop = this.canvas.offsetParent.offsetTop;

      this.rectTip = this.$refs.circleDom;
      this.fontTip = this.$refs.fontTip;
    });
  },
  methods: {
    //   sava\undo\redo
    oprationClickHandle(name) {
      if (name == "save") {
        this.saveItAsImage();
      } else if (name == "redo") {
        this.redo();
      } else if (name == "undo") {
        this.undo();
      }
    },
    saveItAsImage() {
      var image = this.canvas.toDataURL();
      console.log(image);
      //保存到本地
      // window.location.href=image;
    },
    //   绘制完成清空canvas1
    canvas1Mouseup(e) {
      this.ctx1.clearRect(0, 0, this.canvasSize.w, this.canvasSize.h);
    },
    //   绘制圆的过程在canvas1上
    canvas1Mousemove(e) {
      if (this.currentTool === "circle" && this.flag) {
        this.rectTipShow = false;
        this.ctx1.clearRect(0, 0, this.canvasSize.w, this.canvasSize.h);
        this.drawCircle(this.ctx1);
      }
    },
    textColorChange(color) {
      this.textColor = color;
    },
    borderColorChange(color) {
      this.borderColor = color;
    },
    changeTool(name) {
      console.log(name);
      this.currentTool = name;
      if (name === "trash") {
        this.clearCanvas();
        this.currentTool = "pencil";
      }
    },
    mouseDownEventHandler(e) {
      this.flag = true;
      this.beginX = e.pageX - this.canvas.offsetParent.offsetLeft;
      this.beginY = e.pageY - this.canvas.offsetParent.offsetTop;
      //   console.log(this.rectTip);
      this.rectTip.style.border = this.borderColor + " " + "1px solid";
    },
    mouseupEventHandler(e) {
      this.flag = false;

      this.endX = e.pageX - this.canvasLeft;
      this.endY = e.pageY - this.canvasTop;

      // 用于撤销或重做的记录操作历史
      this.historyPush();

      switch (this.currentTool) {
        case "line": {
          this.drawline();
          break;
        }
        case "rect": {
          this.drawRectangle();
          break;
        }
        case "text": {
          this.fontTip.focus();
          break;
        }
        case "circle": {
          this.drawCircle(this.ctx);
          break;
        }
      }
    },
    mouseMoveEventHandler(e) {
      this.ctx.lineWidth = 1;
      switch (this.currentTool) {
        case "pencil": {
          this.drawPencil(e);
          break;
        }
        case "eraser": {
          this.ctx.lineWidth = 15;
          this.drawEraser(e);

          break;
        }
        case "line": {
          this.fakeLineInput(e);
          break;
        }
        case "rect": {
          this.fakeRectangleInput(e);
          break;
        }
        case "text": {
          this.fakeWordsInput(e);
          break;
        }
        case "circle": {
          this.fakeRectangleInput(e);
          break;
        }
      }
    },

    //在画布上画不规则的线
    drawRandomLine(e) {
      //按下鼠标时
      var x = e.pageX - this.canvasLeft;
      var y = e.pageY - this.canvasTop;
      if (this.flag) {
        this.ctx.lineTo(x, y);
        this.ctx.stroke();
      } else {
        this.ctx.beginPath();
        this.ctx.moveTo(x, y);
      }
    },
    drawPencil(e) {
      (this.containerClass = "container_pencil"),
        (this.ctx.strokeStyle = this.borderColor);
      this.drawRandomLine(e);
    },
    drawEraser(e) {
      (this.containerClass = "container_eraser"), (this.ctx.lineCap = "round");
      this.ctx.fillStyle = this.borderColor;
      this.ctx.strokeStyle = this.borderColor;
      this.drawRandomLine(e);
    },
    /**
     * 使用文字工具时，
     * 当鼠标被按下时，可以在画布上拖动一行
     */
    fakeWordsInput(e) {
      const top = e.pageY - this.canvasTop;
      const left = e.pageX - this.canvasLeft;
      if (this.flag) {
        this.fontTipObj = {
          show: true,
          top: this.beginY,
          left: this.beginX,
          width: left - this.beginX,
          height: top - this.beginY,
        };
        //   console.log(this.fontTipObj)
      }
    },
    fakeRectangleInput(e) {
      const left = this.canvasLeft;
      const top = this.canvasTop;
      this.endX = e.pageX - left;
      this.endY = e.pageY - top;

      var borderWidth = 1;
      if (this.flag) {
        this.currentTool === "rect" && (this.rectTipShow = true);
        const x = this.beginX;
        const y = this.beginY;
        let left, top;
        if (this.endX - x < 0 && this.endY - y < 0) {
          left = x - Math.abs(this.endX - x);
          top = y - Math.abs(this.endY - y);
        } else if (this.endX - x < 0) {
          left = x - Math.abs(this.endX - x);
          top = y;
        } else if (this.endY - y < 0) {
          left = x;
          top = y - Math.abs(this.endY - y);
        } else {
          left = x;
          top = y;
        }
        this.rectTip.style.left = left + "px";
        this.rectTip.style.top = top + "px";
        this.rectTip.style.width = Math.abs(this.endX - x) - borderWidth + "px";
        this.rectTip.style.height =
          Math.abs(this.endY - y) - borderWidth + "px";
        console.log(this.flag);
      }
    },

    /**
     * 画线
     */
    fakeLineInput(e) {
      if (this.flag) {
        this.anyLineShow = true;
        this.endX = e.pageX - this.canvasLeft;
        this.endY = e.pageY - this.canvasTop;
        const { degree, x, y, z } = this.getDegreeOfTowPoints(
          { x: this.beginX, y: this.beginY },
          { x: this.endX, y: this.endY }
        );

        this.lineLength = z;
        this.lineLeft = this.beginX + x;
        this.lineTop = this.beginY + y;
        this.lineTransform = degree;
      }
    },

    /**
     * 清空画布上的内容
     */
    clearCanvas() {
      this.ctx.fillStyle = "#FFFFFF";
      var width = this.canvas.offsetWidth;
      var height = this.canvas.offsetHeight;
      this.ctx.fillRect(0, 0, width, height);
    },
    /**
     * 画线的方法
     */
    drawline() {
      this.ctx.beginPath();
      this.ctx.moveTo(this.beginX, this.beginY);
      this.ctx.lineTo(this.endX, this.endY);
      this.ctx.stroke();
      this.anyLineShow = false;
    },

    /**
     * 画矩形的方法e
     */
    drawRectangle() {
      var borderWidth = 1;
      this.ctx.strokeStyle = this.borderColor;
      this.ctx.beginPath();
      this.ctx.strokeRect(
        this.beginX - 1,
        this.beginY - 1,
        this.endX - this.beginX,
        this.endY - this.beginY
      );
      this.rectTip.style.display = "none";

      this.rectTipShow = false;

      this.rectTip.style.width = "0px";
      this.rectTip.style.Height = "0px";
      this.rectTip.style.top = "0px";
      this.rectTip.style.left = "0px";
    },

    /**
     * 画圆形的方法（贝塞尔曲线）
     */

    drawCircle(context) {
      var k = (this.endX - this.beginX) / 0.75 / 2,
        w = (this.endX - this.beginX) / 2,
        h = (this.endY - this.beginY) / 2,
        x = (this.endX + this.beginX) / 2,
        y = (this.endY + this.beginY) / 2;
      if (context) {
        context.strokeStyle = this.borderColor;
        context.beginPath();
        context.moveTo(x, y - h);
        context.bezierCurveTo(x + k, y - h, x + k, y + h, x, y + h);
        context.bezierCurveTo(x - k, y + h, x - k, y - h, x, y - h);
        context.closePath();
        context.stroke();
      }
    },
    /**
     * 输入文字的方法
     */
    drawWords(e) {
      var words = this.textValue.split(/\s/);
      console.log(words);
      if (words.length && words[0]) {
        this.ctx.strokeStyle = this.textColor;
        this.ctx.fillStyle = this.textColor;
        var fontSize = 14;
        this.ctx.font = "14px 宋体";
        words.forEach((item, index) => {
          this.ctx.fillText(
            item,
            this.fontTipObj.left,
            this.fontTipObj.top + 14 + 1 + index * 19
          );
        });

        this.textValue = "";
      }
      this.fontTipObj.show = false;
    },
    /**
     * 记录当前画布（相当于缓存）
     */
    historyPush() {
      this.cStep++;
      if (this.cStep < this.history.length) {
        this.history.length = this.cStep;
      }

      this.history.push(this.canvas.toDataURL());
      console.log("history", this.history.length);
      console.log("cstep", this.cStep);
    },
    /**
     * 撤销
     */
    undo() {
      if (this.cStep >= 0) {
        this.clearCanvas();
        this.cStep--;
        var tempImage = new Image();
        tempImage.src = this.history[this.cStep];
        tempImage.onload = () => {
          this.ctx.drawImage(tempImage, 0, 0);
        };
      }
    },

    /**
     * 重做
     */
    redo() {
      if (this.cStep < this.history.length - 1) {
        this.clearCanvas();
        this.cStep++;
        var tempImage = new Image();
        tempImage.src = this.history[this.cStep];
        tempImage.onload = () => {
          this.ctx.drawImage(tempImage, 0, 0);
        };
      }
    },

    // 计算两点的旋转角度
    getDegreeOfTowPoints(p1, p2) {
      var data = {};
      var width = p2.x - p1.x;
      var height = p2.y - p1.y;
      var arctan = Math.abs(height / width);
      var arc = Math.atan(arctan);
      let x = 0,
        y = 0;

      var angle = (arc / Math.PI) * 180;

      data.z = Math.sqrt(Math.pow(height, 2) + Math.pow(width, 2));

      if (width > 0 && height > 0) {
        y = (data.z * Math.sin(arc)) / 2;
        x = -(data.z - data.z * Math.cos(arc)) / 2;
        data.degree = angle;
      } else if (width < 0 && height > 0) {
        x = -(data.z + data.z * Math.cos(arc)) / 2;
        y = (data.z * Math.sin(arc)) / 2;
        data.degree = 180 - angle;
      } else if (width < 0 && height < 0) {
        x = -(data.z + data.z * Math.cos(arc)) / 2;
        y = -(data.z * Math.sin(arc)) / 2;
        data.degree = 180 + angle;
      } else if (width > 0 && height < 0) {
        x = -(data.z - data.z * Math.cos(arc)) / 2;
        y = -(data.z * Math.sin(arc)) / 2;
        data.degree = -angle;
      }

      return { degree: "rotate(" + data.degree + "deg)", x, y, z: data.z };
    },
    /**
     *
     * 本地上传图片
     */
    uploadImg() {
      var inputObj = document.createElement("input");
      inputObj.type = "file";
      inputObj.accept = "image/*";
      const _this = this;
      inputObj.addEventListener(
        "change",
        function () {
          var file = this.files[0]; //获取input输入的图片
          console.log(this);
          if (!/image\/\w+/.test(file.type)) {
            alert("请确保文件为图像类型");
            return false;
          } //判断是否图片，在移动端由于浏览器对调用file类型处理不同，虽然加了accept = 'image/*'，但是还要再次判断
          var reader = new FileReader();
          reader.readAsDataURL(file); //转化成base64数据类型
          reader.onload = function (e) {
            _this.checkImage(
              _this.canvasSize.w - 20,
              _this.canvasSize.h - 20,
              this.result,
              _this
            );
          };
        },
        false
      );
      // inputObj.id = id;
      inputObj.click();
    },
    /*修改图片尺寸*/
    checkImage(maxWidth, maxHeight, imgData, _this) {
      var img = new Image();
      img.src = imgData;
      img.onload = function () {
        var hRatio;
        var wRatio;
        var Ratio = 1;
        var w = img.width;
        var h = img.height;
        console.log(w, h);
        wRatio = maxWidth / w; /*画布除原  >1 不缩放*/
        hRatio = maxHeight / h; /*小于1 缩放*/
        if (wRatio > 1 && hRatio > 1) {
          Ratio = 1;
        } else if (wRatio > 1 && hRatio < 1) {
          Ratio = hRatio;
        } else if (wRatio < 1 && hRatio > 1) {
          Ratio = wRatio;
        } else if (wRatio < 1 && hRatio < 1) {
          if (wRatio > hRatio) {
            Ratio = hRatio;
          } else Ratio = wRatio;
        }
        w = w * Ratio;
        h = h * Ratio;
        img.height = h;
        img.width = w;
        _this.ctx.drawImage(img, 10, 10, w, h);
        _this.historyPush();
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.toolBar{
    display: flex;
    justify-content: start;
    vertical-align:middle;
}
input[type="radio"]{
    width: 0px;
    height: 0px;
    margin:0;
}
.toolsOptions{
        display: flex;
    justify-content: space-between;
    vertical-align:middle;
    line-height: 1em;
}
.toolsOptions label{
    border: 1px solid #ccc;
    font-weight: normal;
    color: #ffffff;
    border-radius: 5px;
        padding: .3em 1em;
        line-height: 1em;
        cursor: pointer;
        margin-right: .4em;
}
canvas {
  border: 2px dashed gray;
}

.container_pencil {
  cursor: url(images/PencilToolCursor.gif), pointer;
}

.container_eraser {
  cursor: url(images/test.png), pointer;
}

.container_font {
  cursor: crosshair;
}
.speed {
  top: 15px;
}

.ui-selectmenu-text {
  font-size: 14px;
}
.anyLine {
  position: absolute;
  top: auto;
  left: auto;
}
.fontTip {
  background: transparent;
  position: absolute;
}
</style>
