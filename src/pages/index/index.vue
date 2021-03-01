<template>
  <view class="page homepage">
      <cu-custom bgColor="bg-gradual-green">
				<block slot="content">广外北校信息交流平台</block>
      </cu-custom>
    <view>
      <swiper 
        class="card-swiper square-dot" 
        :indicator-dots="true" 
        circular="true" 
        :autoplay="true" 
        interval="5000" 
        duration="500" 
        @change="cardSwiper" 
        indicator-color="#8799a3" 
        indicator-active-color="#39b54a"
      >
        <swiper-item v-for="(item, index) in swiperList" :key="index" :class="cardCur==index?'cur':'' ">
          <view class="swiper-item">
            <image :src="item" mode="aspectFill"></image>
          </view>
        </swiper-item>
      </swiper>
    </view>
    <view class="button-grid">
      <navigator url="/pages/market/index" hover-class="none">
        <button class="nav-button">
          <div class="button-content">
            <div class="button-icon">
              <text class="iconfont icon-wicon-ge-ren-xian-zhi"></text>
            </div>
          </div> 
        </button>
        <p class="button-text">闲置交易</p>
      </navigator>
      <navigator url="/pages/intern/index" hover-class="none">
        <button class="nav-button">
          <div class="button-content">
            <div class="button-icon">
              <text class="iconfont icon-shixi-"></text>
            </div>
          </div>
        </button>
        <p class="button-text">实习共享</p>
      </navigator>
      <navigator url="/pages/express/index" hover-class="none">
        <button class="nav-button">
          <div class="button-content">
            <div class="button-icon">
              <text class="iconfont icon-kuaidi2"></text>
            </div>
          </div>
        </button>
        <p class="button-text">快递代取</p>
      </navigator>
      <navigator url="/pages/other/index" hover-class="none">
        <button class="nav-button">
          <div class="button-content">
            <div class="button-icon">
              <text class="iconfont icon-qita"></text>
            </div>
          </div>
        </button>
        <p class="button-text">其他</p>
      </navigator>
    </view>
      <!-- 链接样式可参考淘宝等商城 -->
    <view class="item-box">
      <text class="item-box-title">—— 上新推荐 ——</text>
      <view class="card-box" v-for="(item, index) in info" :key="index">
        <uni-card
          class="card secondhand-box"
          v-if="item.big_class == '二手'"
          @click="goToDetail(item)"
          :title="item.big_class"
          mode="style"
          :is-shadow="true"
          :thumbnail="item.imageURL"
        >
          <view class="cu-form-group">
            <image class="cu-avatar lg" :src="item.pict"/>
            <text class="cu-item text-center" style="width: 70%;">{{item.title}}</text>
          </view>
        </uni-card>
        <uni-card
          class="card"
          v-if="item.big_class == '其他'"
          @click="goToDetail(item)"
          :title="item.big_class"
          mode="style"
          :is-shadow="true"
          :thumbnail="item.imageURL"
        >
          <view class="cu-form-group">
            <image class="cu-avatar lg" :src="item.pict"/>
            <text class="cu-item text-center" style="width: 70%;">{{item.title}}</text>
          </view>
        </uni-card>

        <uni-card
          class="card"
          v-if="item.big_class == '实习'"
          @click="goToDetail(item)"
          :title="item.jobName" 
          mode="title" 
          :thumbnail="item.pict"
          :is-shadow="true"  
          :extra="item.big_class"
          note="true"
        >
          <text class="description">{{item.jobDescription}}</text>
        </uni-card>
        <uni-card
          class="card"
          v-if="item.big_class == '快递'"
          @click="goToDetail(item)"
          :title="item.express" 
          mode="title" 
          :thumbnail="item.pict"
          :is-shadow="true"  
          :extra="item.big_class"
          note="true"
        >
          <text class="description">{{item.remarks}}</text>
        </uni-card>
      </view>
    </view>
    <!-- 新建按钮 -->
    <view>
      <uni-fab
        :pattern="pattern"
        :content="content"
        :horizontal="horizontal"
        :vertical="vertical"
        :direction="direction"
        @trigger="trigger"
      ></uni-fab>
    </view>
    <view>
      <image
        src="../../static/images/gotop.png"
        class="goTop"
        :hidden="!floorstatus"
        @click="goTop()"
      />
    </view>
    <view class="loading cu-load line-olive" :hidden="!loadMore" :class="!isLoad?'loading':'over'"></view>
    <view class="loading cu-load bg-green" :hidden="!loadAll" :class="!isLoad?'over':'loading'"></view>
  </view>
</template>

<script>
  import {uniCard} from '@dcloudio/uni-ui' 
  // import uniCard from '@dcloudio/uni-ui/lib/uni-card/uni-card.vue' //也可使用此方式引入组件
  import uniFab from "../component/uni-fab.vue";

  var currentPage = 0 // 当前第几页,0代表第一页 
  var pageSize = 4 //每页显示多少数据 

  export default {
    components: {uniCard,uniFab},
    data() {
      return {
        pattern: {
          color: "#7A7E83",
          backgroundColor: "#fff",
          selectedColor: "#D29566",
          buttonColor: "#D29566"
        },
        content: [
        {
          iconPath: "../../static/images/cart-2.png",
          selectedIconPath: "../../static/images/cart-2.png",
          text: "闲置",
          active: false
        },
        {
          iconPath: "../../static/images/intern-2.png",
          selectedIconPath: "../../static/images/intern-2.png",
          text: "实习",
          active: false
        },
        {
          iconPath: "../../static/images/express-2.png",
          selectedIconPath: "../../static/images/express-2.png",
          text: "快递",
          active: false
        },
        {
          iconPath: "../../static/images/others-2.png",
          selectedIconPath: "../../static/images/others-2.png",
          text: "其他",
          active: false
        }
      ],
      horizontal: "right",
      vertical: "bottom",
      direction: "vertical",
        cardCur: 0,
        swiperList: [
          "cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/hbg-1.jpg",
          "cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/hbg-4.jpg",
          "cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/hbg-3.jpg",
          "cloud://cloud-vriuz.636c-cloud-vriuz-1302426019/images/hbg-2.jpg"
        ],
        info: [],
        floorstatus: false,
        // dataList: [], //放置返回数据的数组  
        loadMore: false, //"上拉加载"的变量，默认false，隐藏  
        loadAll: false, //“没有数据”的变量，默认false，隐藏 
        isLoad:false,
      };
    },
    methods: {
    // 获取滚动条当前位置
    onPageScroll(e) {
      // console.log(e);
      if (e.scrollTop > 100) {
        this.floorstatus = true;
      } else {
        this.floorstatus = false;
      }
    },

    //页面上拉触底事件的处理函数
      onReachBottom() {
        // console.log("上拉触底事件")
        let that = this
        if (!that.loadMore) {
          that.loadMore = true, //加载中  
          that.loadAll = false //是否加载完所有数据
          //加载更多，这里做下延时加载
          setTimeout(function() {
            that.getData('qadd')
          }, 2000)
        }
      },
  //访问网络,请求数据  
  getData(mode) {
    if (mode == 'q') {
      currentPage = 0;
      this.info = []
    }
    var that = this;
    //第一次加载数据
    if (currentPage == 1) {
        that.loadMore = true, //把"上拉加载"的变量设为true，显示  
        that.loadAll = false //把“没有数据”设为false，隐藏  
    }
      wx.cloud.init();
      var db = wx.cloud.database({
        env: "cloud-vriuz"
      });
      db.collection("secondhand")
        .skip(currentPage * pageSize) //从第几个数据开始
        .limit(pageSize)  //取几个数据
        .where({})
        .orderBy("Date", "desc")
        .get({})
        .then(res => {
            if (res.data && res.data.length > 0) {
            console.log("请求成功", res.data)
            currentPage++
            //把新请求到的数据添加到dataList里  
            var list = that.info.concat(res.data)
            that.info = list, //获取数据数组    
            that.loadMore = false //把"上拉加载"的变量设为false，显示  
            if (res.data.length < pageSize) {
              that.loadMore = false, //隐藏加载中。。
              that.loadAll = true //所有数据都加载完了
            }
          } else {
              that.loadAll = true, //把“没有数据”设为true，显示  
              that.loadMore = false //把"上拉加载"的变量设为false，隐藏  
          }         
        });
  },

    //回到顶部
    goTop(e) {
      // 一键回到顶部
      if (wx.pageScrollTo) {
        wx.pageScrollTo({
          scrollTop: 0
        });
      } else {
        wx.showModal({
          title: "提示",
          content:
            "当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。"
        });
      }
    },

    trigger(e) {
      console.log(e);
      if (!wx.getStorageSync("info")[0]) {
          wx.switchTab({
        url: "/pages/profile/index"
      });
    }
  
      if ((e.index == 0)) {
        //  this.content[e.index].active= true;
        wx.navigateTo({
          url: "/pages/newMarket/index"
        });
      } else if ((e.index == 1)) {
        //  this.content[e.index].active= true;
        wx.navigateTo({
          url: "/pages/newIntern/index"
        });
      } else if ((e.index == 2)) {
        //  this.content[e.index].active= true;
        wx.navigateTo({
          url: "/pages/newExpress/index"
        });
      } else if ((e.index == 3)) {
        //  this.content[e.index].active= true;
        wx.navigateTo({
          url: "/pages/newOther/index"
        });
      }
    },

    cardSwiper(e) {
      this.cardCur = e.detail.current;
    },
    goToDetail(info) {
      var my_id = info._id;
      if (info.big_class == "二手") {
        uni.navigateTo({ url: "/pages/marketDetail/index?my_id=" + my_id });
      }
      if (info.big_class == "快递") {
        uni.navigateTo({ url: "/pages/expressDetail/index?my_id=" + my_id });
      }
      if (info.big_class == "实习") {
        uni.navigateTo({ url: "/pages/internDetail/index?my_id=" + my_id });
      }
      if (info.big_class == "其他") {
        uni.navigateTo({ url: "/pages/otherDetail/index?my_id=" + my_id });
      }
    }
  },
  onLoad() {
      this.getData('q')
    }
  };
</script>

<style scoped>
  .cu-avatar.lg {
    width: 86rpx;
    height: 84rpx;
    font-size: 2em;
    border-radius: 15rpx;
  }

  .cu-form-group {
    padding: 0;

  }

  .homepage {
    text-align: center;
  }

  .magrith-0{
    margin-top: 0rpx;
    padding-top: 0rpx;
	}
	.card-swiper swiper-item{
    padding: 25rpx 0rpx 50rpx;
	}
  
  .icon-wicon-ge-ren-xian-zhi {
    font-size: 72rpx;
    color: #D29566;
    margin-left: 3rpx;
  }
  .icon-qita {
    font-size: 70rpx;
    color: #5c8d7b;
    margin-left: 8rpx;
  }
  .icon-shixi- {
    font-size: 70rpx;
    color: #3FA9B4;
    margin-left: 10rpx;
  }
  .icon-kuaidi2 {
    font-size: 70rpx;
    color: #f06b5c;
    margin-left: 10rpx;
  }
  
  navigator {
    margin: 30rpx 20rpx;
  }
  .button-grid {
    position: relative;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;
    padding: 5rpx;
    margin: 0rpx 30rpx 10rpx;
  }
  .nav-button {
    position: relative;
    padding: 0;
    width: 90%;
    height: 130rpx;
    outline: none;
    background-color: #f4f5f6;
    border-radius: 40rpx;
    box-shadow:  -6rpx -20rpx 35rpx #e0e0dd, 6rpx 20rpx 25rpx rgba(0,0,0,0.2);
    transition: .13s ease-in-out;
    cursor: pointer;
  }
  .nav-button:active {
    box-shadow: none;  
  }
  .nav-button:active .button-content {
    box-shadow: none;
  }
  .nav-button:active .button-content .button-text,  .nav-button:active .button-content .button-icon {
    transform: translate3d(0rpx, 0rpx, 0rpx)
  }
  .button-content {
    position: relative;
    padding: 20rpx;
    width: 100%;
    height: 100%;
    box-shadow: inset 0rpx -20rpx 0rpx #dddddd, 0rpx -20rpx 0rpx #f4f5f6;
    border-radius: 40rpx;
    transition: .13s ease-in-out;
    z-index: 1;
  }

  .button-icon {
    position: relative;
    display: flex;
    /*grid-column: 1 / span 1;*/
    transform: translate3d(0rpx, -4rpx, 0rpx);
    align-self: start;
    justify-self: end;
    width: 32rpx;
    height: 32rpx;
    transition: .13s ease-in-out;
    margin-top: -48rpx;
  }

  .button-text {
    position: relative;
    transform: translate3d(0rpx, -4rpx, 0rpx);
    margin-top: 28rpx;
    align-self: end;
    text-align: center;
    font-size: 30rpx;
    background-color: #0a0e11;
    color: transparent;
    text-shadow: 2rpx 2rpx 3rpx rgba(255, 255, 255, 0.5);
    -webkit-background-clip: text;
    -moz-background-clip: text;
    background-clip: text;
    transition: .13s ease-in-out;
  }

  .item-box {
    padding: 10rpx 20rpx;
    background-color: white;
    border-radius: 40rpx;
    padding: 20rpx 5rpx;
    margin: 0 20rpx;
  }

  .item-box-title {
    color: #06623b;
    letter-spacing: 8rpx;
    font-style: oblique;
    display: block;
    margin: 25rpx 5rpx;
    clear: both;
  }

  .item-box:after {
    content: "";
    display: block;
    clear: both;
  }
  
  .card-box {
    float: left;
    width: 47.75%;
    margin: 10rpx 5rpx 10rpx 11.3rpx;
  }

  .description {
    display: inline-block;
    height: 163rpx;
    overflow: auto;
  }

  .goTop {
    height: 80rpx;
    width: 80rpx;
    position: fixed;
    bottom: 1rpx;
    background: rgba(247, 247, 247, 0.3);
    right: 0rpx;
    border-radius: 50%;
    z-index: 99;
  }
</style>