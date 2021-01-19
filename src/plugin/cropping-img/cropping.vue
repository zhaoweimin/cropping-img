<template>
  <div class="cropping" @mousemove="mouseMove" @mouseup="mouseUp">
    <!-- 按钮部分 -->
    <div class="btn-line" v-if="!hideBtn">
      <button class="rel" type="primary" v-if="setShow.choose">
        <input class="lucency" type="file" ref="inputer" @input="change" />
        选择文件
      </button>
      <button v-if="setShow.cropping" type="primary" @click="croppingImage">
        生成截图
      </button>
      <button v-if="setShow.export" type="primary" @click="getImage">
        导出
      </button>
    </div>
    <!-- 图片裁剪部分 -->
    <div class="crop-image" :style="{ width: width, height: height }">
      <div class="crop-image-content" v-if="fileUrl">
        <!--  裁剪部分  -->
        <div class="crop-box" ref="cropBox">
          <!-- 第一个图片是底层图 加蒙层 -->
          <img
            ref="cropimg1"
            class="cropimg1"
            :src="fileUrl"
            @load="setPosInfo"
          />
          <!-- 第二个图片用来显示选中的图片 -->
          <img
            class="cropimg2"
            :src="fileUrl"
            :style="{ clip: clip }"
            ref="cropimg2"
          />
          <!--  裁剪框  -->
          <div
            class="main-box"
            :style="{
              width: mainBoxSize.width + 'px',
              height: mainBoxSize.height + 'px',
              top: mainBoxSize.top + 'px',
              left: mainBoxSize.left + 'px',
            }"
            ref="mainBox"
            @mousedown.stop="mouseDown($event, 'mainBox')"
          >
            <div
              id="left-up"
              class="min-box left-up"
              @mousedown.stop="mouseDown($event, 'left-up')"
            ></div>
            <div
              id="up"
              class="min-box up"
              @mousedown.stop="mouseDown($event, 'up')"
            ></div>
            <div
              id="right-up"
              class="min-box right-up"
              @mousedown.stop="mouseDown($event, 'right-up')"
            ></div>
            <div
              id="left"
              class="min-box left"
              @mousedown.stop="mouseDown($event, 'left')"
            ></div>
            <div
              id="right"
              class="min-box right"
              @mousedown.stop="mouseDown($event, 'right')"
            ></div>
            <div
              id="left-down"
              class="min-box left-down"
              @mousedown.stop="mouseDown($event, 'left-down')"
            ></div>
            <div
              id="down"
              class="min-box down"
              @mousedown.stop="mouseDown($event, 'down')"
            ></div>
            <div
              id="right-down"
              class="min-box right-down"
              @mousedown.stop="mouseDown($event, 'right-down')"
            ></div>
          </div>
        </div>

        <!--  预览部分  -->
        <!-- 第一个div用来不显示，用于给html2canvas生成canvas -->
        <div class="hide-preview">
          <div ref="preview">
            <img class="cropimg3" ref="cropimg3" alt="" :src="fileUrl" />
          </div>
        </div>
        <!-- 生成的canvas显示在这 -->
        <div ref="showPreview" class="preview-view"></div>
        <!--  预览部分end  -->
      </div>
    </div>
  </div>
</template>
<script>
import html2canvas from "html2canvas";
export default {
  props: {
    hideBtn: {
      type: Boolean,
      default: false,
    },
    btns: {
      type: [Object, Array],
      default: () => {
        return ["choose", "cropping", "export"];
      },
    },
    width: {
      type: String,
      default: "100%",
    },
    height: {
      type: String,
      default: "auto",
    },
  },
  computed: {
    // 图片显示的位置
    clip: function() {
      let { rect } = this;
      return `rect(${rect.start}px, ${rect.x}px, ${rect.y}px, ${rect.end}px)`;
    },
    setShow: function() {
      let res = {};
      this.btns.forEach((vo) => {
        res[vo] = 1;
      });
      return res;
    },
  },
  data() {
    return {
      fileUrl: "",
      file: "",
      show: false,
      ifKeyDown: false, // 判断鼠标是否按下
      contact: "", // 设置当前拖动的是那个方向的模块,
      posInfo: {},
      rect: {
        start: 0,
        x: 0,
        y: 0,
        end: 0,
      },
      // 裁剪框大小
      mainBoxSize: {
        width: "150",
        height: "150",
        left: "0",
        top: "0",
      },
      startPoint: { x: 0, y: 0 },
    };
  },
  mounted() {
    // this.setPosInfo()
  },
  methods: {
    change() {
      let files = this.$refs.inputer.files; //获取el-input 的files
      // let url = window.URL || window.webkitURL
      // let src = url.createObjectURL(files[0]) // 这里传的是File对象

      // 图片file转base64 由于html2canvas 不能画出跨域的图片 所以这里转出base64
      this.setImage(files[0])
    },
    setImage(arg) {
      let that = this;
      // 判断入参格式
      switch(typeof arg) {
        case 'string':
          if(arg.indexOf('data:image/jpg;base64,')){
              that.fileUrl = arg
          } else {
            this.getCanvasBase64(arg).then(base64 => {
              that.fileUrl = base64
            })
          }

          break;
        case 'object':
          if(arg instanceof File) {
            var reader = new FileReader()
            reader.readAsDataURL(arg)
            reader.onload = function() {
              that.fileUrl = this.result
            }
          } else{
            console.log('请输入图片路径')
          }
          break
        default:
          console.log('请输入图片路径')
      }
      
    },
    // 设置图片数据初始化，图片加载完执行
    setPosInfo() {
      let cropimgElem = this.$refs.cropimg1;
      var cropimgElemWidth = cropimgElem.clientWidth;
      var cropimgElemHeight = cropimgElem.clientHeight;
      var cropimgElemPosition = this.getPosition(cropimgElem);
      var minX = cropimgElemPosition.left; // 待裁剪的图片最小x坐标
      var maxX = cropimgElemPosition.left + cropimgElemWidth; // 待裁剪的图片最大x坐标
      var minY = cropimgElemPosition.top; // 待裁剪的图片最小y坐标
      var maxY = cropimgElemPosition.top + cropimgElemHeight; // 待裁剪的图片最大y坐标

      this.posInfo = {
        minX,
        maxX,
        minY,
        maxY,
      };

      this.setChoice();
      this.show = true;
    },
    // 设置裁剪框的位置
    setChoice() {
      var mainBoxElem = this.$refs.mainBox;
      this.rect = {
        start: mainBoxElem.offsetTop,
        x: mainBoxElem.offsetLeft + mainBoxElem.offsetWidth,
        y: mainBoxElem.offsetTop + mainBoxElem.offsetHeight,
        end: mainBoxElem.offsetLeft,
      };
    },
    mouseDown(e, key) {
      // 设置鼠标点击下去的是那个块
      this.ifKeyDown = true;
      this.contact = key;
      this.startPoint = {
        x: e.clientX,
        y: e.clientY,
        left: this.mainBoxSize.left,
        top: this.mainBoxSize.top,
      };
    },
    mouseMove(e) {
      if (this.ifKeyDown) {
        switch (this.contact) {
          case "mainBox":
            this.mainMove(e);
            break;
          case "right":
            this.rightMove(e);
            break;
          case "up":
            this.upMove(e);
            break;
          case "left":
            this.leftMove(e);
            break;
          case "down":
            this.downMove(e);
            break;
          case "left-up":
            this.leftMove(e);
            this.upMove(e);
            break;
          case "right-up":
            this.rightMove(e);
            this.upMove(e);
            break;
          case "left-down":
            this.leftMove(e);
            this.downMove(e);
            break;
          case "right-down":
            this.rightMove(e);
            this.downMove(e);
            break;
          default:
            break;
        }
        this.setChoice();
      }
    },
    mouseUp() {
      this.ifKeyDown = false;
      this.contact = "";
    },
    getPosition(node) {
      var left = node.offsetLeft;
      var top = node.offsetTop;
      var parent = node.offsetParent;
      while (parent != null) {
        left += parent.offsetLeft;
        top += parent.offsetTop;
        parent = parent.offsetParent;
      }
      return { left: left, top: top };
    },
    mainMove(e) {
      let { startPoint, posInfo, mainBoxSize } = this;
      // 计算鼠标便宜后的left top
      let left = e.clientX - startPoint.x + parseFloat(startPoint.left); // 鼠标x坐标
      let top = e.clientY - startPoint.y + parseFloat(startPoint.top); // 鼠标x坐标
      // 计算最大的left top
      let maxLeft = posInfo.maxX - posInfo.minX - parseFloat(mainBoxSize.width);
      let maxTop = posInfo.maxY - posInfo.minY - parseFloat(mainBoxSize.height);
      // 范围内可以计算偏移
      if (left >= 0 && left < maxLeft) {
        this.mainBoxSize.left = left;
      }
      if (top >= 0 && top < maxTop) {
        this.mainBoxSize.top = top;
      }
    },
    // 右边移动
    rightMove(e) {
      var x = e.clientX; // 鼠标x坐标
      if (x > this.posInfo.maxX || x < this.posInfo.minX) {
        return;
      }
      var mainBoxElem = this.$refs.mainBox; // 裁剪框
      var widthBefore = mainBoxElem.offsetWidth - 2; // 裁剪框变化前的宽度
      var addWidth = x - this.getPosition(mainBoxElem).left - widthBefore; // 鼠标移动后，裁剪框增加的宽度
      var width = widthBefore + addWidth;
      if (width < 1) {
        return;
      }
      // mainBoxElem.style.width = width + 'px' // 裁剪框变化后，设置宽度
      this.mainBoxSize.width = width; // 裁剪框变化后，设置宽度
    },
    // 上边移动
    upMove(e) {
      /**
       * mainDom.scrollTop的值是获取当前div滚动的距离
       * clientY是当前位置距离浏览器顶部的距离
       * posInfo.maxY是cropimg1在页面距离页面顶部的距离
       */
      let mainDom = document.body;
      var y = e.clientY + mainDom.scrollTop; // 鼠标y坐标
      if (y > this.posInfo.maxY || y < this.posInfo.minY) {
        return;
      }
      var mainBoxElem = this.$refs.mainBox; // 裁剪框
      var mainY = this.getPosition(mainBoxElem).top; // 裁剪框相对于屏幕上边的距离
      var addHeight = mainY - y; // 增加的高度
      var heightBefore = mainBoxElem.offsetHeight - 2; // 裁剪框变化前的高度
      var height = heightBefore + addHeight;
      if (height < 1) {
        return;
      }
      this.mainBoxSize.height = height; // 裁剪框变化后，设置高度
      this.mainBoxSize.top = mainBoxElem.offsetTop - addHeight; // 裁剪框相对于父控件的距离
    },
    // 左边移动
    leftMove(e) {
      var x = e.clientX; // 鼠标x坐标
      if (x > this.posInfo.maxX || x < this.posInfo.minX) {
        return;
      }
      var mainBoxElem = this.$refs.mainBox; // 裁剪框
      var mainX = this.getPosition(mainBoxElem).left;
      var widthBefore = mainBoxElem.offsetWidth - 2; // 裁剪框变化前的宽度
      var addWidth = mainX - x; // 鼠标移动后，裁剪框增加的宽度
      var width = widthBefore + addWidth;
      if (width < 1) {
        return;
      }
      this.mainBoxSize.width = width; // 裁剪框变化后，设置宽度
      this.mainBoxSize.left = mainBoxElem.offsetLeft - addWidth; // 裁剪框变化后，设置到父元素左边的距离
    },
    // 下边移动
    downMove(e) {
      /**
       * mainDom.scrollTop的值是获取当前div滚动的距离
       * clientY是当前位置距离浏览器顶部的距离
       * posInfo.maxY是cropimg1在页面距离页面顶部的距离
       */
      let mainDom = document.body;
      var y = e.clientY + mainDom.scrollTop; // 鼠标y坐标
      if (y > this.posInfo.maxY || y < this.posInfo.minY) {
        return;
      }
      var mainBoxElem = this.$refs.mainBox; // 裁剪框
      var heightBefore = mainBoxElem.offsetHeight - 2; // 裁剪框变化前的高度
      var mainY = this.getPosition(mainBoxElem).top; // 裁剪框相对于屏幕上边的距离
      var addHeight = y - heightBefore - mainY; // 增加的高度
      var height = heightBefore + addHeight;
      if (height < 1) {
        return;
      }
      this.mainBoxSize.height = height; // 裁剪框变化后，设置高度
    },
    croppingImage() {
      this.setPreview();
    },
    setPreview() {
      let bodyTop = document.documentElement.scrollTop;
      // let cropimgElem = this.$refs.cropimg1;

      var mainBoxElem = this.$refs.mainBox; // 裁剪框
      var top = mainBoxElem.offsetTop + bodyTop;
      var right = mainBoxElem.offsetLeft + mainBoxElem.offsetWidth;
      var bottom = mainBoxElem.offsetTop + mainBoxElem.offsetHeight;
      var left = mainBoxElem.offsetLeft;

      // 设置 preview 的大小，用来html2canvas生成截图
      var preview = this.$refs.preview;
      preview.style.width = mainBoxElem.offsetWidth + "px";
      preview.style.height = mainBoxElem.offsetHeight + "px";

      // 设置 preview 的图片显示尺寸
      var previewImg = this.$refs.cropimg3;
      // 设置图片大小
      let cropimgElem = this.$refs.cropimg1;
      previewImg.style.width = cropimgElem.clientWidth + "px";
      previewImg.style.height = cropimgElem.clientHeight + "px";
      // 设置显示位置
      previewImg.style.top = -top + "px";
      previewImg.style.left = -left + "px";
      previewImg.style.clip =
        "rect(" +
        top +
        "px, " +
        right +
        "px, " +
        bottom +
        "px, " +
        left +
        "px)";

      // 通过html2canvas 生成裁剪图片
      html2canvas(preview).then((canvas) => {
        this.$refs.showPreview.innerHTML = "";
        // this.$refs.showPreview.appendChild(canvas);
        let oImg = new Image();
        oImg.src = canvas.toDataURL(); // 导出图片
        this.$refs.showPreview.appendChild(oImg); // 将生成的图片添加到body
        // var resultLink = document.createElement("a");
        // resultLink.download = `裁剪结果file_${new Date().getTime()}`;
        // resultLink.href = oImg.src;
        // document.body.appendChild(resultLink);
        // resultLink.click();
        // resultLink.remove();

        this._croppingCanvas = canvas;
        this.getCutImage().then((res) => {
          this.$emit("onCropping", res);
        });
      });
    },
    getCutImage() {
      let { _croppingCanvas } = this;
      return new Promise((resolve) => {
        let base64 = _croppingCanvas.toDataURL();
        let file = this.base64ToFile(base64, `file_${new Date().getTime()}`);
        resolve({ base64, file });
      });
    },
    // base64转 file
    base64ToFile(base64, fileName) {
      let arr = base64.split(","),
        type = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], fileName, { type });
    },
    exportImg() {
      let { _croppingCanvas } = this;
      var url = _croppingCanvas.toDataURL();
      var resultLink = document.createElement("a");
      resultLink.download = `裁剪结果file_${new Date().getTime()}`;
      resultLink.href = url;
      document.body.appendChild(resultLink);
      resultLink.click();
      resultLink.remove();
    },
    getImage() {
      this.exportImg();
    },
    getCanvasBase64(src) {
      let that = this
      return new Promise((resolve, reject) => {
        var image = new Image();
        //至关重要
        image.crossOrigin = ""; //需要解决图片跨域问题 image.crossOrigin = '';
        image.src = src;

        if (src) {
          image.onload = function() {
            resolve(that.getBase64Image(image)); //将base64传给done上传处理
            //document.getElementById("container2").appendChild(image);
          }
        } else {
          reject('请输入路径')
        }
      })
      
    },
    getBase64Image(img, width, height) {
      var canvas = document.createElement("canvas");
      canvas.width = width ? width : img.width;
      canvas.height = height ? height : img.height;
      var ctx = canvas.getContext("2d");
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      var dataURL = canvas.toDataURL();
      return dataURL;
    }
  }
};
</script>

<style lang="scss" scoped>
.rel {
  position: relative;
  overflow: hidden;
}

.btn-line {
  padding: 20px 0;
  text-align: left;

  button {
    margin-right: 10px;
  }
  .lucency {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    opacity: 0;
    overflow: hidden;
  }
}

.hide-preview {
  position: fixed;
  left: 0px;
  top: 0px;
  opacity: 0;
  pointer-events: none;
}
.crop-image {
  user-select: none;
  // border: 1px solid #dedede;
  border-radius: 6px;
  height: 100%;
  width: 100%;
  background: #f2f2f2;
}
/* 裁剪部分 */
.crop-image-content {
  position: relative;
  width: 100%;
  margin-top: 10px;
  text-align: left;
}

.crop-box {
  position: relative;
  display: inline-block;
  width: 49.5%;
}

.crop-image-content .cropimg1 {
  opacity: 0.5;
  // position: absolute;
  max-width: 100%;
  max-height: 100%;
  top: 0;
  left: 0;
  vertical-align: bottom;
}

.crop-image-content .cropimg2 {
  position: absolute;
  top: 0;
  left: 0;
  max-width: 100%;
  max-height: 100%;
  // clip: rect(0, 150px, 150px, 0);
}
/* 预览框 */
.hide-preview {
  opacity: 0;
  // pointer-events: none;
  position: absolute;
  top: 0;
  max-width: 100%;
  max-height: 100%;
}
.hide-preview .cropimg3 {
  position: absolute;
  top: 0;
  left: 0;
}

.crop-image-content .main-box {
  border: 1px solid white;
  position: absolute;
  // top: 0;
  // left: 0;
  cursor: move;
}

.min-box {
  position: absolute;
  height: 8px;
  width: 8px;
  background-color: white;
}

.left-up {
  top: -4px;
  left: -4px;
  cursor: nw-resize;
}

.up {
  left: 50%;
  margin-left: -4px;
  top: -4px;
  cursor: n-resize;
}

.right-up {
  right: -4px;
  top: -4px;
  cursor: ne-resize;
}

.left {
  top: 50%;
  margin-top: -4px;
  left: -4px;
  cursor: w-resize;
}

.right {
  top: 50%;
  margin-top: -4px;
  right: -4px;
  cursor: w-resize;
}

.left-down {
  bottom: -4px;
  left: -4px;
  cursor: sw-resize;
}

.down {
  bottom: -4px;
  left: 50%;
  margin-left: -4px;
  cursor: s-resize;
}

.right-down {
  bottom: -4px;
  right: -4px;
  cursor: se-resize;
}

.preview-view {
  position: absolute;
  left: 50.5%;
  text-align: center;
  right: 0;
  top: 50%;
  transform: translate(0, -50%);
}
</style>
