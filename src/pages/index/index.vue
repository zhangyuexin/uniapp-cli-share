<template>
	<view class="content">
    <view class="draw-img">
      <image :src="shareImage" show-menu-by-longpress class="share-image" :style="{width: '100%',height:canvasHeight+'px'}" @click="preImg"></image>
      <canvasdrawer :painting="painting" @getImage="eventGetImage"></canvasdrawer>
    </view>
    <view class="btn">
      <view @click="eventSave">保存至手机相册</view>
      <view class="cancel">重新生成</view>
    </view>
	</view>
</template>

<script>
import canvasdrawer from "@/components/canvasdrawer/canvasdrawer";
	export default {
		data() {
			return {
        canvasHeight:'',
        shareImage:'',
        painting:{},
        paintinged:{
          "width": 375,
          "height": 555,
          "clear": true,
          "views": [
            {
              "type": "image",
              "url": "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1602478808861&di=2dfd90b69ef34fb417913b2822aabe6e&imgtype=0&src=http%3A%2F%2Fa0.att.hudong.com%2F56%2F12%2F01300000164151121576126282411.jpg",
              "top": 0,
              "left": 0,
              "width": 375,
              "height": 555
            },
            {
              "type": "image",
              "url": "https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1906469856,4113625838&fm=26&gp=0.jpg",
              "top": 200,
              "left": 200,
              "width": 100,
              "height": 100
            },
            {
              "type": "text",
              "content": "文字1",
              "fontSize": 14,
              "color": "red",
              "textAlign": "left",
              "top": 208,
              "left": 50
            },
            {
              "type": "text",
              "content": "文字2",
              "fontSize": 16,
              "color": "yellow",
              "textAlign": "left",
              "top": 300,
              "left": 50
            },
            {
              "type": "text",
              "content": "文字三",
              "fontSize": 18,
              "color": "blue",
              "textAlign": "left",
              "top": 422,
              "left": 163
            }
          ]
        },
      }
    },
    components: {
      canvasdrawer
    },
    async onLoad(option){
       this.getSystem()
       this.eventDraw()
    },
		methods: {
      async getSystem(){
        let that=this
        let res=await uni.getSystemInfo()
        wx.createSelectorQuery().select('.btn').boundingClientRect(function (rect) {
          that.canvasHeight=res[1].windowHeight-rect.height
        }).exec()
      },
      eventDraw() {
        wx.showLoading({
          title: "请稍后",
          mask: true
        });
        this.painting = this.paintinged;
      },
      async eventSave() {
        // 保存图片至本地
        const res = await wx.saveImageToPhotosAlbum({
          filePath: this.shareImage
        });
        if (res.errMsg === "saveImageToPhotosAlbum:ok") {
          wx.showToast({
            title: "保存图片成功",
            icon: "success",
            duration: 2000
          });
        }
      },
      eventGetImage(event) {
        wx.hideLoading();
        this.shareImage = event.tempFilePath;
      },
      preImg(){
        let that=this
        wx.previewImage({
          urls: [that.shareImage] // 需要预览的图片http链接列表
        })
      }
		}
	}
</script>

<style lang="less" scoped>
	.content{
    width: 100%;
    height: 100vh;

    .btn{
      width: 100%;
      height: 98rpx;
      line-height: 98rpx;
      position: fixed;
      left: 0;
      bottom: 0;

      view{
        float: left;
        width: 50%;
        height: 100%;
        background-color: #00A578;
        color: #fff;
        text-align: center;
        font-size: 30rpx;
        font-family: PingFang SC;
        font-weight: 400;
      }

      .cancel{
        background-color: #fff;
        color: #656565;
      }
    }

    .draw-img{
      width: 100%;
    }

  }
</style>
