<template>
  <view>
    <cu-custom bgImage="../../static/images/banner1.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">其他</block>
		</cu-custom>
    <!-- 每条记录  -->
    <view class="margin-top">
      <view
        class="item-card cu-card article"
        v-for="(item, index) in secondhand"
        :key="index"
        @click="goToDetail(item)"
      >
        <view class="cu-item radius shadow">
          <view class="title">
            <view class="text-cut">
              {{item.title}}
              <view class="cu-tag bg-mauve light sm round" style="float: right; margin-top: 36rpx">发布者：{{item.nickname}}</view>
            </view>
          </view>
          <view class="content">
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
      </view>
    </view>

    <view class="loading cu-load line-red" :hidden="!loadMore" :class="!isLoad?'loading':'over'"></view>
    <view class="loading cu-load bg-green" :hidden="!loadAll" :class="!isLoad?'over':'loading'"></view>
    <!-- <button class="cu-btn line-green" @click="load()">加载更多</button> -->
  </view>
</template>

<script>
var currentPage = 0; // 当前第几页,0代表第一页
var pageSize = 4; //每页显示多少数据
export default {
  data() {
    return {
      secondhand: [],
      loadMore: false, //"上拉加载"的变量，默认false，隐藏
      loadAll: false, //“没有数据”的变量，默认false，隐藏
      isLoad:false,
    };
  },
  onLoad() {
    this.getData();
  },
  methods: {
    // 加载数据
    load(){
      let that = this;
      if (!that.loadMore) {
        (that.loadMore = true), //加载中
          (that.loadAll = false); //是否加载完所有数据

        //加载更多，这里做下延时加载
        setTimeout(function() {
          that.getData();
        }, 2000);
      }
    },

    //页面上拉触底事件的处理函数
    onReachBottom() {
      this.load();
    },


    //访问网络,请求数据
    getData() {
      var that = this;
      //第一次加载数据
      if (currentPage == 1) {
        (that.loadMore = true), //把"上拉加载"的变量设为true，显示
          (that.loadAll = false); //把“没有数据”设为false，隐藏
      }

      wx.cloud.init();
      var db = wx.cloud.database({
        env: "cloud-vriuz"
      });
      db.collection("secondhand")
        .skip(currentPage * pageSize) //从第几个数据开始
        .limit(pageSize) //取几个数据
        .where({
          big_class:"其他"
        })
        .orderBy("Date", "desc")
        .get({})
        .then(res => {
          if (res.data && res.data.length > 0) {
            console.log("请求成功", res.data);
            currentPage++;
            //把新请求到的数据添加到dataList里
            var list = that.secondhand.concat(res.data);
            (that.secondhand = list), //获取数据数组
              (that.loadMore = false); //把"上拉加载"的变量设为false，显示

            if (res.data.length < pageSize) {
              (that.loadMore = false), //隐藏加载中。。
                (that.loadAll = true); //所有数据都加载完了
            }
          } else {
            (that.loadAll = true), //把“没有数据”设为true，显示
              (that.loadMore = false); //把"上拉加载"的变量设为false，隐藏
          }
        });
    },


    //点开某条记录，去到新页面
    goToDetail(item) {
      var my_id = item._id;
      uni.navigateTo({ url: "/pages/otherDetail/index?my_id=" + my_id });
    }
  }
};
</script>

<style scoped>
.navbar {
  flex: none;
  display: flex;
  background: #fff;
}

.navbar .item {
  position: relative;
  flex: auto;
  text-align: center;
  line-height: 80rpx;
}

.navbar .item.active {
  color: #247521;
}

.navbar .item.active:after {
  content: "";
  display: block;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 4rpx;
  background: #247521;
}

.cu-item {
  margin-top: 6rpx;
  margin-bottom: 5rpx;
}

.detail-info {
  display: inline-block;
  height: 150rpx;
  overflow: auto;
  color: gray;
}

.tag-box {
  margin-top: 10rpx;
}
</style>