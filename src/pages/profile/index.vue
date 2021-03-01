<template>
  <view>
    <view v-if="isHide">
      <view v-if="canIUse">
        <view class="header">
          <image src="cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/shakehands.jpg" />
        </view>

        <view class="content">
          <view>申请获取以下权限</view>
          <text>获得你的公开信息(昵称，头像等)</text>
        </view>

        <button
          class="bottom"
          type="primary"
          open-type="getUserInfo"
          lang="zh_CN"
          @getuserinfo="bindGetUserInfo"
        >授权登录</button>
      </view>
      <view v-else>请升级微信版本</view>
    </view>
    <view v-else>
      <view class="UCenter-bg" :style="{backgroundImage:'url('+ bingPict +')'}">
        <image :src="pict" style="width:60px;height:60px;border-radius:50%" />
        <view class="margin-top-sm">
          <text>{{nickname}}</text>
        </view>
        <image src="cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/wave.gif" mode="scaleToFill" class="gif-wave">
      </view>
      <view class="cu-list menu card-menu margin-top-xl margin-bottom-xl shadow-lg radius">
        <view class="cu-item arrow"  @tap="showPublished">
          <view>
            <text class="cuIcon-upload text-orange cu-item-icon"></text>
            <text class="text-black cu-item-text text-lg">我的发布</text>
          </view>
        </view>
        <view class="cu-item arrow" @tap="showCollection">
          <view>
            <text class="cuIcon-favorfill text-yellow cu-item-icon"></text>
            <text class="text-black cu-item-text text-lg">我的收藏</text>
          </view>
        </view>
        <view class="cu-item arrow" @tap="showChat">
          <view>
            <text class="cuIcon-messagefill text-olive cu-item-icon"></text>
            <text class="text-black cu-item-text text-lg">我的消息</text>
          </view>
        </view>
      </view>
    </view>
  </view>
</template> 

<script>
export default {
  data() {
    return {
      pict: "",
      nickname: "",
      userId: "",
      canIUse: wx.canIUse("button.open-type.getUserInfo"),
      isHide: false,
      isShowP: true,
      isShowC: false,
      bingURL:
        "https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN",
      bingPict: ""
    };
  },
  onLoad() {
    var that = this;
    //查看是否授权
    wx.getSetting({
      success: function(res) {
        that.isHide = false;
        if (res.authSetting["scope.userInfo"]) {
          wx.getUserInfo({
            success: function(res) {
              that.pict = res.userInfo.avatarUrl;
              that.nickname = res.userInfo.nickName;
              wx.login({
                success: res => {
                  console.log(res.code);
                  wx.request({
                    // 自行补上自己的 APPID 和 SECRET
                    url:
                      "https://api.weixin.qq.com/sns/jscode2session?appid=wx0c21bbfdcef266f5&secret=6c4f9720746524e4b3539086f4fba5d8&js_code=" +
                      res.code +
                      "&grant_type=authorization_code",
                    success: res => {
                      // 获取到用户的 openid
                      console.log("用户的openid:" + res.data.openid);
                      that.userId = res.data.openid;
                      wx.setStorageSync("info", [
                        that.nickname,
                        that.userId,
                        that.pict
                      ]);
                    }
                  });
                }
              });
            }
          });
        } else {
          that.isHide = true;
        }
      }
    });
    //获取必应壁纸
    wx.request({
      url: that.bingURL,
      method: "POST",
      dataType: "json",
      header: {
        "content-type": "application/json; charset=UTF-8"
      },
      success: function(res) {
        console.log(res);
        that.bingPict = "https://cn.bing.com" + res.data.images[0].url;
        // resolve(bingPict);
      },
      fail: function(res) {
        console.log("bing err");
      }
    });
  },
  methods: {
    bindGetUserInfo(e) {
      var that = this;
      if (e.mp.detail.userInfo) {
        //用户按了允许授权按钮
        // 获取到用户的信息了，打印到控制台上看下
        // console.log("用户的信息如下：");
        console.log(e.mp.detail);
        //授权成功后,通过改变 isHide 的值，让实现页面显示出来，把授权页面隐藏起来
        that.isHide = false;
        that.nickname = e.mp.detail.userInfo.nickName;
        that.pict = e.mp.detail.userInfo.avatarUrl;
        wx.getUserInfo({
          success: function(res) {
            that.pict = res.userInfo.avatarUrl;
            that.nickname = res.userInfo.nickName;
            wx.login({
              success: res => {
                console.log(res.code);
                wx.request({
                  // 自行补上自己的 APPID 和 SECRET
                  url:
                    "https://api.weixin.qq.com/sns/jscode2session?appid=wx0c21bbfdcef266f5&secret=6c4f9720746524e4b3539086f4fba5d8&js_code=" +
                    res.code +
                    "&grant_type=authorization_code",
                  success: res => {
                    // 获取到用户的 openid
                    console.log("用户的openid:" + res.data.openid);
                    that.userId = res.data.openid;
                    wx.setStorageSync("info", [
                      that.nickname,
                      that.userId,
                      that.pict
                    ]);
                  }
                });
              }
            });
          }
        });
      } else {
        //用户按了拒绝按钮
        wx.showModal({
          title: "警告",
          content: "您点击了拒绝授权，将无法进入小程序，请授权之后再进入!",
          showCancel: false,
          confirmText: "返回授权",
          success: function(res) {
            // 用户没有授权成功，不需要改变 isHide 的值
            if (res.confirm) {
              console.log("用户点击了“返回授权”");
            }
          }
        });
      }
    },
    showPublished() {
      wx.navigateTo({
        url: "/pages/Mypublished/index?_openid=" + this.userId
      });
    },
    showCollection() {
      wx.navigateTo({
        url: "/pages/Mycollection/index?_openid=" + this.userId
      });
    },
    showChat() {
      wx.navigateTo({
        url: "/pages/Mychat/index?_openid=" + this.userId
      });
    }
  }
};
</script>

<style scoped>
.header {
  margin: 90rpx 0 90rpx 50rpx;
  border-bottom: 1px solid #ccc;
  text-align: center;
  width: 650rpx;
  height: 300rpx;
  line-height: 450rpx;
}

.header image {
  width: 200rpx;
  height: 200rpx;
  border-radius: 50%;
}

.content {
  margin-left: 50rpx;
  margin-bottom: 90rpx;
}

.content text {
  display: block;
  color: #9d9d9d;
  margin-top: 40rpx;
}

.bottom {
  border-radius: 80rpx;
  margin: 70rpx 50rpx;
  font-size: 35rpx;
}

.UCenter-bg {
  background-size: cover;
  height: 450rpx;
  display: flex;
  justify-content: center;
  padding-top: 40rpx;
  overflow: hidden;
  position: relative;
  flex-direction: column;
  align-items: center;
  color: #fff;
  font-weight: 300;
  text-shadow: 0 0 3px rgba(0, 0, 0, 0.3);
}

.UCenter-bg text {
  opacity: 0.8;
}

.UCenter-bg image {
  width: 200rpx;
  height: 200rpx;
}

.UCenter-bg .gif-wave{
  position: absolute;
  width: 100%;
  bottom: 0;
  left: 0;
  z-index: 99;
  mix-blend-mode: screen;  
  height: 100rpx;   
}

.cu-item-text {
  margin-left: 30rpx;
}

.cu-item-icon {
  font-size: 1.35em;
}
</style>