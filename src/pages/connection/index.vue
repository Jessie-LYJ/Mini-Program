<template>
  <view>
    <view v-if="wx_id_s">微信号：{{wx_id_s}}</view>
    <view v-else>
      <view>用户须知</view>
      <view>一些关于隐私的注意点</view>
      <!-- 可以点击隐藏 -->
      <view>
        <input type="text" v-model="wx_id" placeholder="请输入微信号" />
        <input type="text" v-model="test_wxid" placeholder="重新输入验证" />
        <button @click="check_wxid">保存</button>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      userId: "",
      wx_id: "",
      test_wxid: "",
      wx_id_s: ""
    };
  },
  onLoad(option) {
    this.userId = option._openid;
  },
  onShow() {
    var that = this;
    wx.cloud.init();
    var db = wx.cloud.database({
      env: "cloud-vriuz"
    });
    db.collection("information")
      .where({
        _openid: that.userId
      })
      .get()
      .then(function(res) {
        if (res.data.length){
            that.wx_id_s = res.data[0].wx_id;
        }else{
            console.log("no wx_id")
        }   
      });
  },
  methods: {
    check_wxid() {
      var that = this;
      if (that.wx_id == that.test_wxid) {
        wx.cloud.init();
        var db = wx.cloud.database({
          env: "cloud-vriuz"
        });
        db.collection("information")
          .add({
            data: {
            //   _openid: that.userId, 不能设置_openid，它是云服务器根据用户的openID自动设置的
              wx_id: that.wx_id
            }
          })
          .then(res => {
            console.log(res);
            //搞一个modal提示
            wx.showToast({
              title: "保存成功",
              icon: "success",
              duration: 1500
            });
            that.wx_id_s = that.wx_id;
          });
      } else {
        wx.showToast({
          title: "输入结果不一致",
          icon: "none",
          duration: 1500
        });
      }
    }
  }
};
</script>

<style scoped>
</style>