<template>
  <view>
    <cu-custom bgImage="../../static/images/eb1.jpg" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">发布快递</block>
		</cu-custom>
    <view class="cu-form-group">
			<view class="title">宿舍楼：</view>
			<picker mode="selector" @change="bindPickerChange" :value="building" :range="dorms">
        <view class="uni-input">{{dorms[index]}}</view>
      </picker>
		</view>

    <view class="cu-form-group">
			<view class="title">快递公司：</view>
      <input type="text" v-model="express" placeholder="申通快递" name="input">
		</view>

    <view class="cu-form-group">
			<view class="title">取件时间：</view>
			<input placeholder="今晚八点前" name="input" type="text" v-model="fetchTime">
			<text class='cuIcon-time text-grey'></text>
		</view>

    
    <view class="cu-form-group">
			<view class="title">取件数量：</view>
			<input type="text" v-model="parcelAmount" placeholder="3个" name="input">
		</view>

  
    <view class="cu-form-group">
			<view class="title">取件名：</view>
			<input placeholder="美少女" name="input" type="text" v-model="recipient">
			<text class='cuIcon-people text-grey'></text>
		</view>

    <!-- <view class="uni-title">取件尾号:</view>
    <view><input type="text" v-model="tailNumber" placeholder="888" /></view>-->

    <view class="cu-form-group">
			<view class="title">报酬：</view>
			<input placeholder="一杯奶茶" name="input" type="text" v-model="reward">
			<text class='cuIcon-moneybag text-orange'></text>
		</view>

    <view class="cu-form-group align-start">
			<view class="title">备注：</view>
			<textarea maxlength="-1" v-model="remarks" placeholder="例：请放在右边闸机上"></textarea>
		</view>
    
    <button class ="cu-btn bg-orange text-white lg shadow block margin" @click="publish">
      发布 <text class="cuIcon-forwardfill"></text>
    </button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      index: 0,
      building: "云山学1栋",
      dorms: [
        "云山学1栋",
        "云山学2栋",
        "云山学3栋",
        "云山学4栋",
        "云山学5栋",
        "云山学6栋",
        "云山学7栋",
        "云山学8栋",
        "云山学9栋",
        "云山学10栋",
        "云山学11栋",
        "云山学12栋",
        "云山学13栋",
        "云山学14栋",
        "云山学15栋",
        "云山学16栋",
        "云山学17栋",
        "云山学18栋"
      ],
      express: "",
      fetchTime: "",
      parcelAmount: "",
      recipient: "",
      // tailNumber:"",
      reward: "",
      // teLNumber:"",
      remarks: "",
      userOpenId: "",
      nickname: "",
      pict: "",
      postDate: "",
      postTime: ""
    };
  },
  onLoad(option) {
    var value = wx.getStorageSync("info");
    this.nickname = value[0];
    this.userOpenId = value[1];
    this.pict = value[2];
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

      this.postDate = `${year}${month}${day}` / 1;
      this.postTime = `${hour}:${min}`;
    },
    bindPickerChange(e) {
      console.log("picker发送选择改变，携带值为", e.target.value);
      this.index = e.target.value;
      this.building = this.dorms[this.index];
    },
    publish() {
      this.getDate();
      var that = this;
      if (
        that.building &&
        that.express &&
        that.fetchTime &&
        that.parcelAmount &&
        that.recipient &&
        that.reward &&
        that.remarks
      ) {
        wx.cloud.init({ env: "cloud-vriuz" });
        var db = wx.cloud.database();
        db.collection("secondhand").add({
          data: {
            big_class: "快递",
            Date: that.postDate,
            Time: that.postTime,
            building: that.building,
            express: that.express,
            fetchTime: that.fetchTime,
            parcelAmount: that.parcelAmount,
            recipient: that.recipient,
            reward: that.reward,
            remarks: that.remarks,
            _openid: that.userOpenId,
            nickname: that.nickname,
            pict: that.pict
          }
        });
        wx.navigateBack();
      } else {
        wx.showToast({
          title: "请填写全部信息",
          icon: "none"
        });
      }
    }
  }
};
</script>

<style scoped>
.cu-form-group textarea {
  height: 14em;
}
</style>