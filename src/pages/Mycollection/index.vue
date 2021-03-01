<template>
  <view>
       <cu-custom bgImage="../../static/images/othersbg.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">我的收藏</block>
		</cu-custom>
    <view v-if="my_collection.length == 0" class="text-gray text-bold text-center" id="msg">空空如也~</view>
    <view v-else>
    <view
      class="item-card cu-card article"
      v-for="(item, index) in my_collection"
      :key="index"
    >

			<view class="cu-item radius shadow" v-if="item.big_class == '二手'">
				<view class="title">
          <view class="text-cut">
            {{item.title}}
            <view class="cu-tag bg-red light xl round" style="float: right; margin-top: 28rpx"><text class="cuIcon-delete" @tap="remove(index, item)"></text></view>
            <view class="cu-tag bg-mauve light sm round margin-right-lg" style="float: right; margin-top: 36rpx">发布者：{{item.nickname}}</view>
          </view>
        </view>
				<view class="content" @tap="goToDetail(item)">
					<image :src="item.imageURL" mode="aspectFill"></image>
					<view class="desc">
						<view class="text-content">
              <view class="item-detail">{{item.remarks}}</view>
              <view class="item-detail">价格:{{item.price}}</view>
              <view class="item-detail">发布时间：{{item.Date}}--{{item.Time}}</view>
            </view>
						<view>
							<view class="cu-tag bg-orange light sm round">已用时长:{{item.usedTime}}</view>
							<view class="cu-tag bg-green light sm round">新旧程度:{{item.newOld}}</view>
						</view>
					</view>
				</view>
			</view>

      <view class="cu-item radius shadow" v-if="item.big_class == '其他'">
          <view class="title">
            <view class="text-cut">
              {{item.title}}
              <view class="cu-tag bg-red light xl round" style="float: right; margin-top: 28rpx"><text class="cuIcon-delete" @tap="remove(index, item)"></text></view>
              <view class="cu-tag bg-mauve light sm round margin-right-lg" style="float: right; margin-top: 36rpx">发布者：{{item.nickname}}</view>
            </view>
          </view>
          <view class="content" @tap="goToDetail(item)">
            <image :src="item.imageURL" mode="aspectFill"></image>
            <view class="desc">
              <view class="detail-info">
                <view class="item-detail">{{item.remarks}}</view>
              </view>
              <view class="tag-box">
                <view class="cu-tag bg-red light sm round">发布日期：{{item.Date}}--{{item.Time}}</view>
              </view>
            </view>
          </view>
        </view>
      
      <view class="cu-item radius shadow" v-if="item.big_class == '实习'">
				<view class="title">
          <view class="text-cut">
            <view class="cu-avatar round" :style="{backgroundImage: 'url('+ item.pict +')'}"></view>
            <text class="margin-left">{{item.jobName}}</text>
            <view class="cu-tag bg-red light xl round" style="float: right; margin-top: 28rpx"><text class="cuIcon-delete" @tap="remove(index, item)"></text></view>
            <view class="cu-tag bg-orange light sm round margin-right-lg" style="float: right; margin-top: 36rpx">{{item.field}}</view>
          </view>
        </view>
				<view class="content" @tap="goToDetail(item)">
					<view class="desc">
						<view class="text-content detail-box">
              <view class="detail-info">
                <view class="item-detail">{{item.jobDescription}}</view>
                <view class="item-detail">【简历投递方式:{{item.resumeSend}}】</view>
              </view>
            </view>
						<view class="tag-box">
							<view class="cu-tag bg-green light sm round">{{item.workUnit}}</view>
              <view class="cu-tag bg-blue light sm round">{{item.workPlace}}</view>
              <view class="cu-tag bg-red light sm round">实习工资:{{item.salary}}</view>
              <view class="cu-tag bg-mauve light sm round">发布者：{{item.nickname}}</view>
						</view>
            <view class="time-box">
              <view class="line-gray" style="margin-top: 5rpx;">{{item.Date}}--{{item.Time}}</view>
            </view>
					</view>
				</view>
			</view>

      <view class="cu-item shadow" v-if="item.big_class == '快递'">
				<view class="cu-list menu-avatar">
					<view class="cu-item">
						<view class="cu-avatar round" :style="{backgroundImage: 'url('+ item.pict +')'}"></view>
						<view class="content" style="marfin-left: -50rpx;">
							<view class="text-bold" style="marfin-left: -50rpx;">{{item.express}}</view>
						</view>
            <view class="cu-tag bg-red light xl round margin-right" style="float: right;"><text class="cuIcon-delete" @tap="remove(index, item)"></text></view>
					</view>
				</view>
				<view class="text-content"  @tap="goToDetail(item)">
          <view class="item-detail detail-info-express padding-left-lg">
            <view>详情：{{item.remarks}}</view>
            <view>取件名：{{item.recipient}}</view>
            <view>报酬：{{item.reward}}</view>
          </view>
				</view>
				<view class="text-gray text-sm text-right padding">
					<view class="cu-tag bg-olive light sm round">{{item.building}}</view>
          <view class="cu-tag bg-cyan light sm round">{{item.express}}</view>
          <view class="cu-tag bg-pink light sm round">取{{item.parcelAmount}}件</view>
          <view class="cu-tag bg-orange light sm round">取件时间：{{item.fetchTime}}</view>
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
      userId: "",
      my_collection: []
    };
  },
  onLoad(option) {
    this.userId = option._openid;
    //拿到用户的openid，来确定是Mycollection中的哪一条记录
  },
  onShow() {
    var that = this;
    wx.cloud.init({
      env: "cloud-vriuz"
    });
    var db = wx.cloud.database();
    db.collection("Mycollection")
      .where({ _openid: that.userId })
      .get()
      .then(res => {
        console.log(res.data);
        that.my_collection = res.data;
      });
  },
  methods: {
    goToDetail(item) {
      var my_id = item._id;
      if (item.big_class == "二手") {
        uni.navigateTo({ url: "/pages/marketDetail/index?my_id=" + my_id });
      }
      if (item.big_class == "快递") {
        uni.navigateTo({ url: "/pages/expressDetail/index?my_id=" + my_id });
      }
      if (item.big_class == "实习") {
        uni.navigateTo({ url: "/pages/internDetail/index?my_id=" + my_id });
      }
      if (item.big_class == "其他") {
        uni.navigateTo({ url: "/pages/otherDetail/index?my_id=" + my_id });
      }
    },
    remove(index, item) {
      var goodId = item._id;
      var that = this;
      wx.cloud.init({
        env: "cloud-vriuz"
      });
      var db = wx.cloud.database();
      db.collection("Mycollection")
        .doc(goodId)
        .remove()
        .then(res => {
          console.log(res.data);
        });
      this.my_collection.splice(index, 1);
    }
  }
};
</script>

<style scoped>
.detail-info {
  display: inline-block;
  height: 138rpx;
  overflow: auto;
}

.tag-box, .time-box {
  margin-top: 15rpx;
}

.time-box {
  text-align: right;
}

.cu-card.article>.cu-item {
  padding-bottom: 13rpx;
}

.cu-item {
  margin-top: 5rpx;
  margin-bottom: 5rpx;
}

.detail-info-express {
  display: inline-block;
  height: 180rpx;
  overflow: auto;
}
#msg {
  margin-top:20px;
  margin-bottom: 8px;
}
</style>
