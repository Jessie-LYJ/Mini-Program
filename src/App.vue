<script>
import Vue from 'vue'
export default {
  onLaunch: function() {
    console.log("App Launch");
    // 登录
    wx.login({
      success: res => {
        // 发送 res.code 到后台换取 openId, sessionKey, unionId
      }
    });
    // 获取用户信息
    wx.getSetting({
      success: res => {
        if (res.authSetting["scope.userInfo"]) {
          // 已经授权，可以直接调用 getUserInfo 获取头像昵称，不会弹框
          wx.getUserInfo({
            success: res => {
              // 可以将 res 发送给后台解码出 unionId
              console.log(res)
              // 由于 getUserInfo 是网络请求，可能会在 Page.onLoad 之后才返回
              // 所以此处加入 callback 以防止这种情况
              if (this.userInfoReadyCallback) {
                this.userInfoReadyCallback(res);
              }
            }
          });
        }
      }
    });
      uni.getSystemInfo({
				success: function(e) {
					// #ifndef MP
					Vue.prototype.StatusBar = e.statusBarHeight;
					if (e.platform == 'android') {
						Vue.prototype.CustomBar = e.statusBarHeight + 50;
					} else {
						Vue.prototype.CustomBar = e.statusBarHeight + 45;
					};
					// #endif

					// #ifdef MP-WEIXIN || MP-QQ
					Vue.prototype.StatusBar = e.statusBarHeight;
					let capsule = wx.getMenuButtonBoundingClientRect();
					if (capsule) {
						Vue.prototype.Custom = capsule;
						// Vue.prototype.capsuleSafe = uni.upx2px(750) - capsule.left + uni.upx2px(750) - capsule.right;
						Vue.prototype.CustomBar = capsule.bottom + capsule.top - e.statusBarHeight;
					} else {
						Vue.prototype.CustomBar = e.statusBarHeight + 50;
					}
					// #endif		
				}
			});
    Vue.prototype.ColorList = [{
					title: '嫣红',
					name: 'red',
					color: '#e54d42'
				},
				{
					title: '桔橙',
					name: 'orange',
					color: '#f37b1d'
				},
				{
					title: '明黄',
					name: 'yellow',
					color: '#fbbd08'
				},
				{
					title: '橄榄',
					name: 'olive',
					color: '#8dc63f'
				},
				{
					title: '森绿',
					name: 'green',
					color: '#39b54a'
				},
				{
					title: '天青',
					name: 'cyan',
					color: '#1cbbb4'
				},
				{
					title: '海蓝',
					name: 'blue',
					color: '#0081ff'
				},
				{
					title: '姹紫',
					name: 'purple',
					color: '#6739b6'
				},
				{
					title: '木槿',
					name: 'mauve',
					color: '#9c26b0'
				},
				{
					title: '桃粉',
					name: 'pink',
					color: '#e03997'
				},
				{
					title: '棕褐',
					name: 'brown',
					color: '#a5673f'
				},
				{
					title: '玄灰',
					name: 'grey',
					color: '#8799a3'
				},
				{
					title: '草灰',
					name: 'gray',
					color: '#aaaaaa'
				},
				{
					title: '墨黑',
					name: 'black',
					color: '#333333'
				},
				{
					title: '雅白',
					name: 'white',
					color: '#ffffff'
				},
			]
  },
  onShow: function() {
    console.log("App Show");
  },
  onHide: function() {
    console.log("App Hide");
  }
};
</script>

<style>
  /*每个页面公共css */
	@import "colorui/main.css";
	@import "colorui/icon.css";
	@import "colorui/animation.css";
	@import "common/iconfont.css";

  page {
	background-color: #f5f5f5;
  }
</style>
