<template>
  <view>
    <cu-custom bgColor="bg-gradual-pink" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">消息列表</block>
		</cu-custom>
    <view>
      <view class="cu-bar bg-white solid-bottom margin-top">
				<view class="action">
					<text class="cuIcon-title text-orange"></text>我收到的信息
				</view>
			</view>
      <view class="cu-list menu-avatar" v-if="messageReceiver.length !== 0">
				<view 
          class="cu-item" 
          :class="modalName=='move-box-'+ index?'move-cur':''" 
          v-for="(item, index) in messageReceiver" 
          :key="index"
          @touchstart="ListTouchStart" 
          @touchmove="ListTouchMove" 
          @touchend="ListTouchEnd" 
          :data-target="'move-box-' + index"
          @tap="goToChat(item)"
        >
					<view class="cu-avatar lg bg-white" style="background-image: url('../../static/images/m2.png')"></view>
					<view class="content cu-form-group">
						<view class="text-black">{{item._id.sender_Nickname}}</view>
					</view>
          <text class="cuIcon-noticefill text-olive cu-item-icon lg margin-right-lg"></text>
					<view class="move">
						<view class="bg-red">查看详情</view>
					</view>
				</view>
			</view>
      <view v-else class="text-gray text-bold text-center" id="msg2" >暂无消息</view>
    </view>

    <view>
      <view class="cu-bar bg-white solid-bottom margin-top">
				<view class="action">
					<text class="cuIcon-title text-orange"></text>我发送的信息
				</view>
			</view>
      <view class="cu-list menu-avatar" v-if="messageSender.length !== 0">
				<view 
          class="cu-item" 
          :class="modalName=='move-box-'+ index?'move-cur':''" 
          v-for="(item, index) in messageSender"
          :key="index" 
          :data-target="'move-box-' + index"
          @tap="goToChat(item)"
        >
					<view class="cu-avatar lg bg-white" style="background-image: url('../../static/images/m3.png')"></view>
					<view class="content cu-form-group">
						<view class="text-black">{{item._id.sender_Nickname}}</view>
					</view>
          <text class="cuIcon-comment text-pink cu-item-icon margin-right-lg"></text>
				</view>
			</view>
      <view v-else class="text-gray text-bold text-center" id="msg2" >您还未发送过消息~</view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userOpenId: "",
      nickname: "",
      pict: "",
      messageReceiver: "",
      messageSender: "",
      modalName: null,
      listTouchStart: 0,
			listTouchDirection: null,
    };
  },
  onLoad() {
    var value = wx.getStorageSync("info");
    this.nickname = value[0];
    this.userOpenId = value[1];
    this.pict = value[2];

    var that = this;

    wx.cloud.init();
    var db = wx.cloud.database({
      env: "cloud-vriuz"
    });
    const $ = db.command.aggregate;
    db.collection("message")
      .aggregate()
      .match({
        receiver: that.userOpenId
      })
      .group({
        _id: {
          his_openid: "$sender",
          sender_Nickname: "$sender_Nickname",
          his_pict: "$sender_pick"
        }
      })
      .end()
      .then(res => {
        that.messageReceiver = res.list;
      });

    wx.cloud.init();
    var db = wx.cloud.database({
      env: "cloud-vriuz"
    });
    db.collection("message")
      .aggregate()
      .match({
        sender: that.userOpenId
      })
      .group({
        _id: {
          his_openid: "$receiver",
          sender_Nickname: "$receiver_Nickname",
          // his_pict:"$sender_pick"
        }
      })
      .end()
      .then(res => {
        console.log(res.list);
        that.messageSender = res.list;
      });
  },
  methods: {
    goToChat(item) {
      console.log("gotochat");

      var his_openid = item._id.his_openid;
      var his_nickname = item._id.sender_Nickname;
      var his_pict = item._id.his_pict;

      uni.navigateTo({
        url:
          "/pages/chat/index?&his_openid=" +
          his_openid +
          "&his_nickname=" +
          his_nickname +
          "&his_pict=" +
          his_pict +
          ""
      });
    },
    	// ListTouch触摸开始
			ListTouchStart(e) {
				this.listTouchStart = e.touches[0].pageX
			},

			// ListTouch计算方向
			ListTouchMove(e) {
				this.listTouchDirection = e.touches[0].pageX - this.listTouchStart > 0 ? 'right' : 'left'
			},

			// ListTouch计算滚动
			ListTouchEnd(e) {
				if (this.listTouchDirection == 'left') {
					this.modalName = e.currentTarget.dataset.target
				} else {
					this.modalName = null
				}
				this.listTouchDirection = null
			},
  }
};
</script>

<style scoped>

.item-nickname{
  text-align: left;
  margin:10px 3px 5px 3px;
}

#msg1,#msg2,#msg3,#msg
{
  margin-top:20px;
  margin-bottom: 8px;
}

.cu-item-icon {
  font-size: 1.3em;
}
</style>