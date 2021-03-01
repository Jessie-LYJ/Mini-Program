<template>
  <view>
    <cu-custom bgImage="../../static/images/banner2.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">发布其他</block>
		</cu-custom>

    <view class="cu-form-group">
      <view class="title">内容类别:</view>
      <input placeholder="安利种草/失物招领/帮填问卷" v-model="category" name="input" />
    </view>

    <view class="cu-form-group">
      <view class="title">标题：</view>
      <input type="text" v-model="title" placeholder="寻找失物" />
      <text class="cuIcon-write text-orange"></text>
    </view>

    <view class="cu-form-group align-start">
      <view class="title">具体内容：</view>
      <textarea maxlength="-1" v-model="remarks" placeholder="具体内容"></textarea>
    </view>

    <view class="cu-bar bg-white">
      <view class="action">图片上传</view>
      <view class="action">
					<text class="text-gray">{{imgList.length}}/1</text>
				</view>
    </view>
    <view class="cu-form-group">
      <!--<img :src="tempImagePath" @click="addImages()" style="width: 40px; height: 40px;" />-->
      <view class="grid col-4 grid-square flex-sub">
					<view class="bg-img" v-for="(item,index) in imgList" :key="index" @tap="ViewImage" :data-url="imgList[index]">
            <image :src="imgList[index]" mode="aspectFill"></image>
						<view class="cu-tag bg-red" @tap.stop="DelImg" :data-index="index">
							<text class='cuIcon-close'></text>
						</view>
					</view>
					<view class="solids" @tap="addImages" v-if="imgList.length<1">
						<text class='cuIcon-cameraadd'></text>
					</view>
				</view>
    </view>
    <button class="cu-btn bg-gradual-green light lg shadow block margin" @click="publish">
      <text class="text-white">发布</text>
      <text class="cuIcon-forwardfill"></text>
    </button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      category: "",
      title: "",
      remarks: "",
      userOpenId: "",
      nickname: "",
      pict: "",
      postDate: "",
      postTime: "",
      imgList: []
    };
  },
  onLoad(option) {
    var value = wx.getStorageSync("info");
    this.nickname = value[0];
    this.userOpenId = value[1];
    this.pict = value[2];
  },
  methods: {
    getDate() {
      const date = new Date();
      let year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDate();
      let hour = date.getHours();
      let min = date.getMinutes();

      year = year;
      month = month > 9 ? month : "0" + month;
      day = day > 9 ? day : "0" + day;
      hour = hour > 9 ? hour : "0" + hour;
      min = min > 9 ? min : "0" + min;

      this.postDate = `${year}${month}${day}` / 1;
      this.postTime = `${hour}:${min}`;
    },
    addImages() {
      // var that = this;
      uni.chooseImage({
        count: 1,
        sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
				sourceType: ['album'], //从相册选择
        success:(res) => {
          // var tempFilePath = res.tempFilePaths[0];
          // that.tempImagePath = tempFilePath;
          if (this.imgList.length != 0) {
						this.imgList = this.imgList.concat(res.tempFilePaths)
					} else {
						this.imgList = res.tempFilePaths
					}
        }
      });
    },
    ViewImage(e) {
			uni.previewImage({
				urls: this.imgList,
				current: e.currentTarget.dataset.url
			});
		},
    DelImg(e) {
				uni.showModal({
					title: '发布闲置图片',
					content: '确定要换掉这张图片吗？',
					cancelText: '再看看',
					confirmText: '确定',
					success: res => {
						if (res.confirm) {
							this.imgList.splice(e.currentTarget.dataset.index, 1)
						}
					}
				})
			},
    publish() {
      this.getDate();
      var that = this;
      var split_paths = this.imgList[0].split("/");
      wx.cloud.init({ env: "cloud-vriuz" });
      wx.cloud
        .uploadFile({
          cloudPath: "images/" + split_paths[split_paths.length - 1],
          filePath: this.imgList[0]
        })
        .then(function(res) {
          if (
            res.fileID && 
            that.category && 
            that.title && 
            that.remarks
          ) {
            var db = wx.cloud.database();
            db.collection("secondhand").add({
              data: {
                big_class: "其他",
                Date: that.postDate,
                Time: that.postTime,
                imageURL: res.fileID,
                category: that.category,
                title: that.title,
                remarks: that.remarks,
                _openid: that.userOpenId,
                nickname: that.nickname,
                pict: that.pict
              }
            });
            wx.navigateBack();
          } else {
            wx.showToast({
              title: "请填写全部信息",
              icon: "none"
            });
          }
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.cu-form-group textarea {
  height: 18em;
}
</style>