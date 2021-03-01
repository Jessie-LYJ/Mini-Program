<template>
  <view class="market-page">
    <cu-custom bgImage="../../static/images/mb1.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">闲置交易</block>
		</cu-custom>
    <!-- 顶部分类导航栏 -->
    <view class="navbar cu-bar bg-white">
      <text
        v-for="(item,index) in navbar"
        :data-idx="index"
        :class="{item, active :currentTab==index}"
        :key="index"
        @click="navbarTap"
      >{{item}}</text>
      <!-- 启动筛选器的按钮 -->
      <button class="cu-btn line-brown shadow margin-right-sm" @tap="changeFilter" data-target="DrawerModalR">
        <text class="cuIcon-filter text-brown"></text>筛选
      </button>
    </view>

    <!-- 筛选器页面，价格，日期等 -->
    <view class="cu-modal drawer-modal justify-end" :class="modalName=='DrawerModalR'?'show':''" @tap="cancelFilter">
			<view class="cu-dialog basis-lg" @tap.stop="" :style="[{top:CustomBar+'px',height:'calc(100vh - ' + CustomBar + 'px)'}]">
				<view class="cu-list menu text-left">
					<view class="cu-item">
            <view class="content">
              <view class="cu-bar bg-white">
                <view class="title">价格区间：</view>
              </view>
              <view>
                <view class="cu-form-group">
                  <text class="text-brown">大于</text>
                  <input v-model="filterPriceStart" type="text" required name id placeholder="大于" class="margin-left-sm bg-gray text-center"/>
                </view>
                <view class="cu-form-group">
                  <text class="text-brown">小于</text>
                  <input v-model="filterPriceEnd" type="text" required name id placeholder="小于" class="margin-left-sm bg-gray text-center"/>
                </view>
              </view>
            </view>
          </view>
          <view class="cu-item">
            <view class="content">
              <view class="cu-bar bg-white">
                <view class="title">日期：</view>
              </view>
              <view>
                <view class="cu-form-group">
                  <text class="text-brown">起始日期：</text>
                  <picker
                    mode="date"
                    :value="start_date"
                    :start="startDate"
                    :end="endDate"
                    @change="bindStartDateChange"
                  >
                    <view class="piker">{{start_date}}</view>
                  </picker>
                </view>
                <view class="cu-form-group">
                  <text class="text-brown">截止日期：</text>
                  <picker
                    mode="date"
                    :value="end_date"
                    :start="startDate"
                    :end="endDate"
                    @change="bindEndDateChange"
                  >
                    <view class="piker">{{end_date}}</view>
                  </picker>
                </view>
              </view>
            </view>
          </view>
          <button class="cu-btn block bg-brown shadow margin" @click="getData('f')">确定</button>
          <button class="cu-btn block bg-brown light shadow margin" @click="cancelFilter">取消</button>
        </view>
      </view>
    </view>
    <!-- 每条记录  -->
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
						<view class="text-content">
              <view class="item-detail">{{item.remarks}}</view>
              <view class="item-detail">价格:{{item.price}}</view>
              <view class="item-detail">发布时间：{{item.Date}}--{{item.Time}}</view>
            </view>
						<view>
							<view class="cu-tag bg-orange light sm round">已用时长:{{item.usedTime}}</view>
							<view class="cu-tag bg-green light sm round">崭新程度:{{item.newOld}}</view>
						</view>
					</view>
				</view>
			</view>
    </view>

    <!-- 载入提示 -->
    <view class="loading cu-load line-red" :hidden="!loadMore" :class="!isLoad?'loading':'over'"></view>
    <view class="loading cu-load bg-green" :hidden="!loadAll" :class="!isLoad?'over':'loading'"></view>

    <!-- 加载更多按钮，启动筛选后出现 -->
    <button class="cu-btn block line-gray" :hidden="!filter" @click="load()">加载更多</button>
  </view>
</template>

<script>
var currentPage = 0; // 当前第几页,0代表第一页
var pageSize = 4; //每页显示多少数据
export default {
  data() {
    const currentDate = this.getDate({
      format: true
    });
    const lastYear = this.getDate("one_year_ago");

    return {
      loadMore: false, //"上拉加载"的变量，默认false，隐藏
      loadAll: false, //“没有数据”的变量，默认false，隐藏
      filter: false, //当前是加载全部还是加载筛选
      CustomBar: this.CustomBar,
      modalName: null,
      isLoad:false,
      start_date: lastYear,
      end_date: currentDate,
      navbar: ["全部", "数码", "学习", "生活"],
      currentTab: 0,
      filterShow: false,
      filterPriceStart: "",
      filterPriceEnd: "",
      secondhand: []
    };
  },
  onLoad() {
    this.getData("q");
  },
  computed: {
    startDate() {
      return this.getDate("start");
    },
    endDate() {
      return this.getDate("end");
    }
  },
  methods: {
    // 加载数据
    load() {
      let that = this;
      if (!that.loadMore) {
        (that.loadMore = true), //加载中
          (that.loadAll = false); //是否加载完所有数据

        if (!that.filter) {
          //如果没筛选时加载全部
          //加载更多，这里做下延时加载
          setTimeout(function() {
            that.getData("qadd");
          }, 2000);
        } else {
          //如果是筛选后加载
          setTimeout(function() {
            that.getData("fadd");
          }, 2000);
        }
      }
    },

    //页面上拉触底事件的处理函数
    onReachBottom() {
      console.log("上拉触底事件");
      this.load();
    },

    bindStartDateChange: function(e) {
      this.start_date = e.target.value;
    },
    bindEndDateChange: function(e) {
      this.end_date = e.target.value;
    },
    getDate(type) {
      const date = new Date();
      let year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDate();

      if (type === "start") {
        year = year - 60;
      } else if (type === "end") {
        year = year + 2;
      } else if (type === "one_year_ago") {
        year = year - 1;
      }

      month = month > 9 ? month : "0" + month;
      day = day > 9 ? day : "0" + day;

      return `${year}-${month}-${day}`;
    },

    //访问网络,请求数据
    getData(mode) {
      // 不同模式控制加载什么数据
      if (mode == "q") {
        //重新加载全部
        currentPage = 0;
        this.secondhand = [];
        this.filter = false;
      } else if (mode == "f") {
        //重新加载筛选
        currentPage = 0;
        this.secondhand = [];
        this.filter = true;
        this.filterShow = false;
        this.modalName = null;
      } else if (mode == "qadd") {
        //加载更多全部
        this.filter = false;
      } else {
        //加载更多筛选
        this.filter = true;
      }

      var compare_start_date =
        (this.start_date.substring(0, 4) +
          this.start_date.substring(5, 7) +
          this.start_date.substring(8, 10)) /
        1;

      var compare_end_date =
        (this.end_date.substring(0, 4) +
          this.end_date.substring(5, 7) +
          this.end_date.substring(8, 10)) /
        1;

      var that = this;
      wx.cloud.init();
      var db = wx.cloud.database({
        env: "cloud-vriuz"
      });
      const _ = db.command;
      //第一次加载数据
      if (currentPage == 1) {
        (that.loadMore = true), //把"上拉加载"的变量设为true，显示
          (that.loadAll = false); //把“没有数据”设为false，隐藏
      }

      //控制条件
      var condition = {};
      if (!that.filter) {
        if (that.currentTab == 0) {
          //全部版块
          condition = {
            big_class: "二手"
          };
        } else {
          condition = {
            //其他版块
            big_class: "二手",
            category: that.navbar[that.currentTab]
          };
        }
      } else {
        if (that.currentTab == 0) {
          //如果再全部版块加筛选条件
          if (that.filterPriceStart && !that.filterPriceEnd) {
            condition = {
              big_class: "二手",
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.gte(parseFloat(that.filterPriceStart))
            };
          } else if (!that.filterPriceStart && that.filterPriceEnd) {
            condition = {
              big_class: "二手",
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.lte(parseFloat(that.filterPriceEnd))
            };
          } else if (!that.filterPriceStart && !that.filterPriceEnd) {
            condition = {
              big_class: "二手",
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              )
            };
          } else {
            condition = {
              big_class: "二手",
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.gte(parseFloat(that.filterPriceStart)).and(
                _.lte(parseFloat(that.filterPriceEnd))
              )
            };
          }
        } else {
          //如果在其他版块加筛选条件
          if (that.filterPriceStart && !that.filterPriceEnd) {
            condition = {
              big_class: "二手",

              category: that.navbar[that.currentTab],
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.gte(parseFloat(that.filterPriceStart))
            };
          } else if (!that.filterPriceStart && that.filterPriceEnd) {
            condition = {
              big_class: "二手",
              category: that.navbar[that.currentTab],
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.lte(parseFloat(that.filterPriceEnd))
            };
          } else if (!that.filterPriceStart && !that.filterPriceEnd) {
            condition = {
              big_class: "二手",
              category: that.navbar[that.currentTab],
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              )
            };
          } else {
            condition = {
              big_class: "二手",
              category: that.navbar[that.currentTab],
              Date: _.gte(parseFloat(compare_start_date)).and(
                _.lte(parseFloat(compare_end_date))
              ),
              price: _.gte(parseFloat(that.filterPriceStart)).and(
                _.lte(parseFloat(that.filterPriceEnd))
              )
            };
          }
        }
      }

      db.collection("secondhand")
        .skip(currentPage * pageSize) //从第几个数据开始
        .limit(pageSize) //取几个数据
        .where(condition)
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

    // 顶部导航条发生改变
    navbarTap(e) {
      this.currentTab = e.currentTarget.dataset.idx;
      this.getData("q");
    },

    //筛选按钮点击,显示或隐藏筛选框
    changeFilter(e) {
      // if (this.filterShow) {
      //   this.filterShow = false;
      // } else {
      //   this.filterShow = true;
      // }
      this.modalName = e.currentTarget.dataset.target
    },

    //隐藏筛选器
    cancelFilter(e) {
      // this.filterShow = false;
      this.modalName = null
    },

    //点开某条记录，去到新页面
    goToDetail(item) {
      var my_id = item._id;
      uni.navigateTo({ url: "/pages/marketDetail/index?my_id=" + my_id });
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
  margin-top: 5rpx;
  margin-bottom: 5rpx;
}

.cu-form-group picker::after {
  display: none;
}
</style>