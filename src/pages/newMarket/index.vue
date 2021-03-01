<template>
  <view>
    <cu-custom bgImage="../../static/images/mb2.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">发布闲置</block>
		</cu-custom>
    <form>
      <view class="cu-form-group">
        <view class="title">物品名称:</view>
        <input placeholder="高级英语上册" v-model="title" name="input" />
      </view>
      <view class="cu-form-group">
        <view class="title">类别:</view>
        <picker @change="bindPickerChange" :value="category" :range="types">{{types[index]}}</picker>
        <text class="cuIcon-sort text-orange"></text>
      </view>
      <view class="cu-form-group">
        <view class="title">价格(单位￥):</view>
        <input type="number" name id="price" v-model="price" placeholder="5" />
        <text class="cuIcon-moneybag text-orange"></text>
      </view>
      <view class="cu-form-group">
        <view class="title">已用时长:</view>
        <input type="text" name id="usedTime" v-model="usedTime" placeholder="半年" />
        <text class="cuIcon-time text-orange"></text>
      </view>

      <view class="cu-form-group">
        <text class="title">崭新程度:</text>
      </view>
      <view class="slider">
        <slider :value="index" @change="sliderChange" show-value block-size=18 activeColor="#D29566" block-color="#D29566" backgroundColor="#f0ece3"/>
      </view>

      <view class="cu-form-group align-start">
        <view class="title">备注:</view>
        <textarea
          maxlength="-1"
          v-model="remarks"
          placeholder="基本全新，只写了几页"
        ></textarea>
      </view>
      
      <view class="cu-bar bg-white">
        <view class="action">图片上传</view>
        <view class="action">
					<text class="text-gray">{{imgList.length}}/1</text>
				</view>
      </view>
      <view class="cu-form-group">
        <!--<img :src="tempImagePath" @click="addImages" style="width: 40px; height: 40px;" />-->
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
    </form>
    <button class="cu-btn bg-brown lg shadow block margin" @click="publish">
      <text class="text-white">发布</text>
      <text class="cuIcon-forwardfill"></text>
    </button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      index: 0,
      category: "数码",
      price: "",
      usedTime: "",
      newOld: 50,
      types: ["数码", "学习", "生活"],
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
  // onLoad() {
  //   this.tempImagePath = this.tempFilePath;
  //   imagepath=true
  // },

  methods: {
    sliderChange(e) {
      console.log("value 发生变化：" + e.detail.value);
      this.newOld = e.target.value;
    },

    bindPickerChange(e) {
      console.log("picker发送选择改变，携带值为", e.target.value);
      this.index = e.target.value;
      this.category = this.types[this.index];
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

      this.price = this.price / 1;
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
            that.remarks &&
            that.price &&
            that.usedTime &&
            that.newOld
          ) {
            var db = wx.cloud.database();
            db.collection("secondhand").add({
              data: {
                big_class: "二手",
                Date: that.postDate,
                Time: that.postTime,
                imageURL: res.fileID,
                category: that.category,
                title: that.title,
                remarks: that.remarks,
                price: that.price,
                usedTime: that.usedTime,
                newOld: that.newOld,
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

<style scoped>
.slider {
  background-color: white;
  padding:5rpx;
}
</style>