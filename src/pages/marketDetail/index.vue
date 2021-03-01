<template>
  <view>
    <cu-custom bgImage="../../static/images/mb1.jpg" :isBack="true">
      <block slot="backText">返回</block>
      <block slot="content">闲置交易</block>
    </cu-custom>
    <!-- 每条记录 -->
    <view class="item-card" v-for="(item, index) in secondhand" :key="index">
      <view>
        <view class="img-container">
          <image class="comment-img" :src="item.imageURL" />
        </view>
        <view class="cu-form-group">
          <image class="cu-avatar round xl" :src="item.pict" />
          <view class="text-content margin-bottom margin-top-sm">{{item.nickname}}</view>
        </view>
        <view class="action">
          <view class="cu-form-group">
            <view class="title">物品名称：{{item.title}}</view>
          </view>
          <view class="cu-form-group">
            <view class="title">已用时长：{{item.usedTime}}</view>
          </view>
          <view class="cu-form-group">
            <view class="title">崭新程度：{{item.newOld}}</view>
          </view>
          <view class="cu-form-group">
            <view class="cu-item">
            <view class="title">详情：</view>
            <view class="content padding-tb-sm">
            <view>{{item.remarks}}</view>
            </view>
          </view>
          </view>
          
        </view>
      </view>
      <button class="cu-btn bg-brown shadow" v-if="userOpenId" @click="collect(item)">
        收藏
        <view class="cuIcon-favorfill"></view>
      </button>

      <button class="cu-btn bg-brown shadow" v-if="userOpenId" @click="goToChat(item)">
        发起对话
        <view class="cuIcon-communityfill"></view>
      </button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      my_id: "",
      secondhand: "",
      userOpenId: ""
    };
  },
  onLoad(option) {
    this.my_id = option.my_id;
    var value = wx.getStorageSync("info");
    this.userOpenId = value[1];
  },
  onShow() {
    var that = this;
    wx.cloud.init();
    var db = wx.cloud.database({
      env: "cloud-vriuz"
    });
    db.collection("secondhand")
      .where({
        _id: that.my_id
      })
      .get()
      .then(function(res) {
        console.log(res);
        that.secondhand = res.data;
      });
  },
  methods: {
    collect(item) {
      if (!this.userOpenId) {
        wx.switchTab({
          url: "/pages/profile/index"
        });
      }
      var that = this;
      wx.cloud.init();
      var db = wx.cloud.database({
        env: "cloud-vriuz"
      });
      db.collection("Mycollection")
        .where({
          good_id: item._id,
          _openid: that.userOpenId
        })
        .get()
        .then(res => {
          console.log(res.data);
          if (res.data.length > 0) {
            wx.showToast({
              title: "收藏过了哦",
              icon: "none",
              duration: 1000 //持续的时间
            });
          } else {
            db.collection("Mycollection")
              .add({
                data: {
                  good_id: item._id,
                  _openid: that.userOpenId,
                  big_class: item.big_class,
                  imageURL: item.imageURL,
                  category: item.category,
                  title: item.title,
                  remarks: item.remarks,
                  ori_id: item._openid,
                  nickname: item.nickname,
                  pict: item.pict
                }
              })
              .then(res => {
                // console.log(res);
                wx.showToast({
                  title: "收藏成功",
                  icon: "success",
                  duration: 1500 //持续的时间
                });
              });
          }
        });
    },
    goToChat(item) {
      console.log("gotochat");

      var my_id = item._id;
      var his_openid = item._openid;
      var his_nickname = item.nickname;
      var his_pict = item.pict;

      // uni.navigateTo({url: "/pages/market/index"})
      uni.navigateTo({
        url:
          "/pages/chat/index?my_id=" +
          my_id +
          "&his_openid=" +
          his_openid +
          "&his_nickname=" +
          his_nickname +
          "&his_pict=" +
          his_pict +
          ""
      });
    }
  }
};
</script>

<style lang="css" scoped>
.pict {
  width: 60px;
  height: 60px;
  border-radius: 50%;
}
#first {
  margin: 0 10px 10px 50px;
}
button {
  margin: 10px 40px 20px 40px;
}
</style>