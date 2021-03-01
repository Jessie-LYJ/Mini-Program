<template>
  <view>
      <cu-custom bgImage="../../static/images/ib1.png" bgColor="bg-gradual-blue" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">实习共享</block>
		</cu-custom>
    <!-- 每条记录 -->
    <view class="item-card" v-for="(item, index) in secondhand" :key="index">


          <view class="padding-xl radius shadow shadow-lg bg-white margin-top">
    
      <view class="cu-item">
			<image class="cu-avatar margin-left xl round" :src="item.pict"></image>
      
      <view class="text-content margin-bottom margin-left margin-top-sm">{{item.nickname}}</view>
      </view>
        <view class="cu-tag bg-blue">工作领域:{{item.field}}</view>

          <view class="action">
              <view class="cu-form-group">
              <view class="title">工作名称:{{item.jobName}}</view>
              <text class='cuIcon-news text-blue'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">工作单位:{{item.workUnit}}</view>
              <text class='cuIcon-discover text-blue'></text>
            </view>

            <view class="cu-form-group">
              <view class="title">工作地点:{{item.workPlace}}</view>
              <text class='cuIcon-location text-blue'></text>
            </view>

             <view class="cu-form-group">
              <view class="title">实习工资:{{item.salary}}</view>
              <text class='cuIcon-sponsorfill text-blue'></text>
            </view>

            <view class="cu-form-group">
				      <view class="title">简历投递方式:{{item.resumeSend}}</view>
              <text class='cuIcon-post text-blue'></text>
			      </view> 

            <!-- <view class="cu-form-group" id="jobdescription">  
              <view class="title">职位描述:</view>

             
              <textarea maxlength="-1" disabled="disabled" 
              placeholder="{{item.jobDescription}}">
              </textarea>

              <text class='cuIcon-text text-blue'></text>
              
            </view> -->
              <view class="cu-form-group">
               <view class="title">职位描述:</view>
              <text class='cuIcon-text text-blue'></text>
            </view>

             <view class="content padding-tb-sm">
							<view>{{item.jobDescription}}</view>
              </view>
  

             

			     </view>
      </view>

      <button  class= "cu-btn bg-blue shadow margin-bottom-xl"  v-if="userOpenId" @click="collect(item)">
        收藏 <view class="cuIcon-favorfill"></view>  
      </button>

      <button  class= "cu-btn bg-blue shadow margin-bottom-xl"  v-if="userOpenId" @click="goToChat(item)">
       发起对话<view class='cuIcon-communityfill'></view>
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
                  big_class: item.big_class,
                  jobName: item.jobName,
                  field: item.field,
                  workUnit: item.workUnit,
                  workPlace: item.workPlace,
                  jobDescription: item.jobDescription,
                  salary: item.salary,
                  resumeSend: item.resumeSend,
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

#jobdescription{
  height: 100px;
}

</style>