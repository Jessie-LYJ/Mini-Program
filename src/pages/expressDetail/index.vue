<template>
  <view>
    <cu-custom bgColor="bg-gradual-orange" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">快递代取</block>
		</cu-custom>
    <!-- 每条记录 -->
    <view class="item-card" v-for="(item, index) in secondhand" :key="index">
      <view class="padding-xl radius shadow shadow-lg bg-white margin-top">
    
      <view class="cu-item">
			<image class="cu-avatar margin-left round xl" :src="item.pict"></image>
      
      <view class="text-content margin-bottom margin-left margin-top-sm">{{item.nickname}}</view>
      </view>
        <view class="cu-tag bg-orange">宿舍楼：{{item.building}}</view>
          <view class="action">
              <view class="cu-form-group">
              <view class="title">快递公司：{{item.express}}</view>
              <text class='cuIcon-deliver_fill text-orange'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">取件时间：{{item.fetchTime}}</view>
              <text class='cuIcon-countdown text-orange'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">取件数量：{{item.parcelAmount}}</view>
              <text class='cuIcon-check text-orange'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">取件名：{{item.recipient}}</view>
              <text class='cuIcon-profile text-orange'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">报酬：{{item.reward}}</view>
              <text class='cuIcon-recharge text-orange'></text>
            </view>

            <view class="cu-form-group">
				      <view class="title">详情：{{item.remarks}}</view>
              <text class='cuIcon-form text-orange'></text>
			      </view>  
			     </view>
      </view>

        <button  class= "cu-btn bg-red  shadow"  v-if="userOpenId" @click="collect(item)">
         收藏 <view class="cuIcon-favorfill"></view>  
      </button>

      <button  class= "cu-btn bg-red  shadow"  v-if="userOpenId" @click="goToChat(item)">
        发起对话
        <view class='cuIcon-communityfill'></view>
        </button>

    </view>

      <!-- <view class="item-detail">宿舍楼：{{item.building}}</view>
      <view class="item-detail">快递公司：{{item.express}}</view>
      <view class="item-detail">详情：{{item.remarks}}</view>
      <view class="item-detail">取件时间：{{item.fetchTime}}</view>
      <view class="item-detail">取件数量：{{item.parcelAmount}}</view>
      <view class="item-detail">取件名：{{item.recipient}}</view>
      <view class="item-detail">报酬：{{item.reward}}</view>
      <view>
        <image class="pict" :src="item.pict" />
      </view>
      <view class="item-detail">发布者：{{item.nickname}}========</view> -->


    

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
                  big_class: item.big_class,
                  building: item.building,
                  express: item.express,
                  fetchTime: item.fetchTime,
                  parcelAmount: item.parcelAmount,
                  recipient: item.recipient,
                  reward: item.reward,
                  remarks: item.remarks,
                  _openid: that.userOpenId,
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
/* .pict {
  width: 60px;
  height: 60px;
  border-radius: 50%;
} */

button{
  margin-top:20px;
  margin-left:15px;
}
</style>