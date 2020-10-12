<template>
    <canvas canvas-id="canvasdrawer"
        :style="{width:width+'px',height:height+'px'}"
        class="board"
        v-if="showCanvas">
    </canvas>
</template>

<script>

export default {
  props: {
    painting: {
      type: Object,
      default: {}
    }
  },
  data(){
    return {
      showCanvas: false,
      width: 100,
      height: 100,
      index: 0,
      imageList: [],
      tempFileList: [],
      isPainting: false,
      ctx: null
    }
  },
  watch: {
    painting(newVal, oldVal) {
      if (!this.isPainting) {
        if (JSON.stringify(newVal) !== JSON.stringify(oldVal)) {
          if (newVal && newVal.width && newVal.height) {
            this.showCanvas = true;
            this.isPainting = true;
            this.readyPigment();
          }
        } else {
          if (newVal && newVal.mode !== "same") {
            this.$emit("getImage", {
              errMsg: "canvasdrawer:samme params"
            });
          }
        }
      }
    }
  },
  methods : {
    readyPigment() {
      const {
        width,
        height,
        views
      } = this.painting;
      this.width = width;
      this.height = height;

      const inter = setInterval(() => {
        if (this.ctx) {
          clearInterval(inter);
          this.ctx.clearActions();
          this.ctx.save();
          this.getImageList(views);
          this.downLoadImages(0);
        }
      }, 100);
    },
    getImageList(views) {
      const imageList = [];
      for (let i = 0; i < views.length; i++) {
        if (views[i].type === "image") {
          imageList.push(views[i].url);
        }
      }
      this.imageList = imageList;
    },
    downLoadImages(index) {
      
      const {
        imageList,
        tempFileList
      } = this;
      console.log(index,imageList.length)
      if (index < imageList.length) {
        this.getImageInfo(imageList[index]).then(file => {
          tempFileList.push(file);
          this.tempFileList = tempFileList;
          this.downLoadImages(index + 1);
        });
      } else {
        this.startPainting();
      }
    },
    startPainting() {
      const {
        tempFileList,
        painting: {
            views
        }
      } = this;
      for (let i = 0, imageIndex = 0; i < views.length; i++) {
        if (views[i].type === "image") {
          this.drawImage({
            ...views[i],
            url: tempFileList[imageIndex]
          });
          imageIndex++;
        } else if (views[i].type === "text") {
            if (!this.ctx.measureText) {
              wx.showModal({
                title: "提示",
                content: "当前微信版本过低，无法使用 measureText 功能，请升级到最新微信版本后重试。"
              });
              this.$emit("getImage", {
                errMsg: "canvasdrawer:version too low"
              });
              return;
            } else {
              this.drawText(views[i]);
            }
          } else if (views[i].type === "rect") {
            this.drawRect(views[i]);
          }
      }
      
      this.ctx.draw(false, () => {
        const system = wx.getSystemInfoSync().system
        if (/ios/i.test(system)) {
          this.saveImageToLocal()
        } else {
          // 延迟保存图片，解决安卓生成图片错位bug。
          setTimeout(() => {
            this.saveImageToLocal()
          }, 800)
        }
      });
    },
    drawImage(params) {
      this.ctx.save();
      const {
        url,
        top = 0,
        left = 0,
        width = 0,
        height = 0,
        borderRadius = 0
      } = params;
      this.ctx.drawImage(url, left, top, width, height);
      this.ctx.restore();
    },
    drawText(params) {
      this.ctx.save();
      const {
        MaxLineNumber = 2,
        breakWord = false,
        color = "black",
        content = "",
        fontSize = 16,
        top = 0,
        left = 0,
        lineHeight = 20,
        textAlign = "left",
        width,
        bolder = false,
        textDecoration = "none"
      } = params;
      this.ctx.beginPath();
      this.ctx.setTextBaseline("top");
      this.ctx.setTextAlign(textAlign);
      this.ctx.setFillStyle(color);
      this.ctx.setFontSize(fontSize);
      if (!breakWord) {
        this.ctx.fillText(content, left, top);
        this.drawTextLine(left, top, textDecoration, color, fontSize, content);
      } else {
          let fillText = "";
          let fillTop = top;
          let lineNum = 1;
          for (let i = 0; i < content.length; i++) {
              fillText += [content[i]];
              if (this.ctx.measureText(fillText).width > width) {
                if (lineNum === MaxLineNumber) {
                  if (i !== content.length) {
                    fillText = `${fillText.substring(0, fillText.length - 1)}...`;
                    this.ctx.fillText(fillText, left, fillTop);
                    this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
                    fillText = "";
                    break;
                  }
                }
                this.ctx.fillText(fillText, left, fillTop);
                this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
                fillText = "";
                fillTop += lineHeight;
                lineNum++;
              }
            }
          this.ctx.fillText(fillText, left, fillTop);
          this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
        }
      this.ctx.restore();
      if (bolder) {
        this.drawText({
          ...params,
          left: left + 0.3,
          top: top + 0.3,
          bolder: false,
          textDecoration: "none"
        });
      }
    },
    drawTextLine(left, top, textDecoration, color, fontSize, content) {
      if (textDecoration === "underline") {
        this.drawRect({
          background: color,
          top: top + fontSize * 1.2,
          left: left - 1,
          width: this.ctx.measureText(content).width + 3,
          height: 1
        });
      } else if (textDecoration === "line-through") {
        this.drawRect({
          background: color,
          top: top + fontSize * 0.6,
          left: left - 1,
          width: this.ctx.measureText(content).width + 3,
          height: 1
        });
      }
    },
    drawRect(params) {
      this.ctx.save();
      const {
        background,
        top = 0,
        left = 0,
        width = 0,
        height = 0
      } = params;
      this.ctx.setFillStyle(background);
      this.ctx.fillRect(left, top, width, height);
      this.ctx.restore();
    },
    async getImageInfo(url) {
      const res = await wx.getImageInfo({
        src: url
      });
      if (res.errMsg === "getImageInfo:ok") {
        return res.path;
      } else {
        this.$emit("getImage", {
            errMsg: "canvasdrawer:download fail"
        });
        return new Error("getImageInfo fail");
      }
    },
    async saveImageToLocal() {
      const {
        width,
        height
      } = this;
      const res = await wx.canvasToTempFilePath({
        x: 0,
        y: 0,
        width,
        height,
        canvasId: "canvasdrawer"
      }, this);
      if (res.errMsg === "canvasToTempFilePath:ok") {
        this.showCanvas = false;
        this.isPainting = false;
        this.imageList = [];
        this.tempFileList = [];
        this.$emit("getImage", {
          tempFilePath: res.tempFilePath,
          errMsg: "canvasdrawer:ok"
        });
      } else {
        this.$emit("getImage", {
          errMsg: "canvasdrawer:fail"
        });
      }
    }
  },
  onReady() {
    this.ctx = wx.createCanvasContext("canvasdrawer", this);
  }
}
</script>
<style lang="less">
  .board {
    position: fixed;
    top: 2000rpx;
  }
</style>
