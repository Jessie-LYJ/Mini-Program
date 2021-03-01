<template>
  <view class="chat-room">
    <cu-custom bgColor="bg-gradual-green" :isBack="true">
      <block slot="backText">返回</block>
      <block slot="content">{{his_nickname}}</block>
    </cu-custom>

  <view class="cu-chat">
    <view class="item-card" v-for="(item, index) in allmessage" :key="index">
      <view class="cu-item self" v-if="item.sender == userOpenId">
        <view class="main">
          <view class="content bg-green shadow">
            <text>{{item.message}}</text>
          </view>
        </view>
        <view class="content">
          <view class="cu-avatar radius avatar-diy" :style="{backgroundImage:'url('+ pict +')'}"></view>
        </view>
        <view class="date">{{item.sendTime}}</view>
      </view>

      <view class="cu-item" v-else>
        <view class="content">
          <view class="cu-avatar radius avatar-diy" :style="{backgroundImage:'url('+ item.sender_pict +')'}"></view>
        </view>
				<view class="main">
					<view class="content shadow">
						<text>{{item.message}}</text>
					</view>
				</view>
				<view class="date">{{item.sendTime}}</view>
			</view>
    </view>
  </view>

    <view class="cu-bar foot input" :style="[{bottom:InputBottom+'px'}]">
        <input 
          class="solid-bottom" 
          v-model="my_message" 
          :adjust-position="false" 
          :focus="false" 
          maxlength="300" 
          cursor-spacing="10" 
          @focus="InputFocus" 
          @blur="InputBlur"
          placeholder="请在此输入"
        >
			<button class="cu-btn bg-green shadow" @click="sendMessage()">发送</button>
		</view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      my_message: "",
      groupid: "",
      my_id: "",
      his_openid: "",
      his_nickname: "",
      his_pict: "",
      nickname: "",
      userOpenId: "",
      pict: "",
      sendTime: "",
      sendTimeTS: "",
      allmessage: [],
      isInit: false,
      InputBottom: 0,
    };
  },
  onLoad(option) {
    this.my_id = option.my_id;
    this.his_openid = option.his_openid;
    this.his_nickname = option.his_nickname;
    this.his_pict = option.his_pict;

    var value = wx.getStorageSync("info");
    this.nickname = value[0];
    this.userOpenId = value[1];
    this.pict = value[2];

    var groupid1 = this.userOpenId + this.his_openid;
    var groupid2 = this.his_openid + this.userOpenId;

    var that = this;
    wx.cloud.init();
    var db = wx.cloud.database({
      env: "cloud-vriuz"
    });
    const _ = db.command;
    db.collection("message")
      .where({
        groupid: _.eq(groupid1).or(_.eq(groupid2))
      })
      .orderBy("sendTimeTS", "asc")
      .watch({
        onChange: function(snapshot) {
          // console.log("docs's changed events", snapshot.docChanges);
          if (!that.isInit) {
            console.log("query result snapshot after the event", snapshot.docs);
            that.allmessage = snapshot.docs;

            that.isInit = true;
          } else {
            console.log(snapshot.docChanges[0].doc);
            var list = that.allmessage.concat(snapshot.docChanges[0].doc);
            that.allmessage = list;
          }
        },
        onError: function(err) {
          console.error("the watch closed because of error", err);
        }
      });
      console.log(this.allmessage)
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

      this.sendTime = `${year}-${month}-${day} ${hour}:${min}`;

      var timestamp = Date.parse(new Date());
      this.sendTimeTS = timestamp / 1000;
    },

    sendMessage() {
      this.getDate();
      this.groupid = this.userOpenId + this.his_openid;
      var that = this;
      wx.cloud.init({ env: "cloud-vriuz" });
      var db = wx.cloud.database();
      if (that.my_message) {
        db.collection("message").add({
          data: {
            groupid: that.groupid,
            sender: that.userOpenId,
            sender_Nickname: that.nickname,
            sender_pict: that.pict,
            receiver: that.his_openid,
            receiver_Nickname: that.his_nickname,
            message: that.my_message,
            sendTime: that.sendTime,
            sendTimeTS: that.sendTimeTS
          }
        });
      } else {
        wx.showToast({
          title: "不能发送空白信息",
          icon: "none",
          duration: 800 //持续的时间
        });
      }
      this.my_message = "";
    },
    InputFocus(e) {
			this.InputBottom = e.detail.height
		},
		InputBlur(e) {
			this.InputBottom = 0
		},
  }
};
</script>

<style scoped>
.chat-room {
  padding-bottom: 100rpx;
}
.avatar-diy {
  width: 80rpx;
  height: 75rpx;
}
</style>