<template>
  <view>
    <cu-custom bgImage="../../static/images/ib2.png" :isBack="true">
      <block slot="backText">返回</block>
			<block slot="content">发布实习</block>
		</cu-custom>
    <view class="cu-form-group">
			<view class="title">工作名称:</view>
			<input placeholder="内容运营实习生" type="text" v-model="jobName" name="input" >
		</view>
  
    <view class="cu-form-group">
			<view class="title">工作领域:</view>
			<input placeholder="教育" type="text"  v-model="field" name="input" >
		</view>

    <view class="cu-form-group">
			<view class="title">工作单位:</view>
			<input placeholder="网易公司" type="text"  v-model="workUnit" name="input" >
		</view>

    <view class="cu-form-group">
			<view class="title">工作地点:</view>
			<input placeholder="广州" type="text"  v-model="workPlace"  name="input" >
		</view>


    <view class="cu-form-group align-start">
			<view class="title">职位描述:</view>
			<textarea maxlength="-1" v-model="jobDescription" placeholder="1. 英语相关专业 2.协助项目进行内容运营"></textarea>
    </view>

    <!-- <view class="uni-title">实习时长:</view>
    <view><input type="text" v-model="internship" placeholder="一周3-5天" /></view>-->

    <view class="cu-form-group">
			<view class="title">实习工资:</view>
			<input placeholder="面议" type="text"  v-model="salary"  name="input" >
		</view>

    <view class="cu-form-group">
			<view class="title">简历投递方式:</view>
			<input placeholder="邮箱：1235@qq.com" type="text"  v-model="resumeSend"  name="input" >
		</view>

    <button class ="cu-btn bg-gradual-blue lg shadow block margin" @click="publish">
      发布 <text class="cuIcon-forwardfill"></text>
    </button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      jobName: "",
      field: "",
      workUnit: "",
      workPlace: "",
      jobDescription: "",
      salary: "",
      resumeSend: "",
      userOpenId: "",
      nickname: "",
      pict: "",
      postDate:"",
      postTime:"",
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
    publish() {
      this.getDate();
      var that = this;
      if (
        that.jobName &&
        that.field &&
        that.workUnit &&
        that.workPlace &&
        that.jobDescription &&
        that.salary &&
        that.resumeSend
      ) {
        wx.cloud.init({ env: "cloud-vriuz" });
        var db = wx.cloud.database();
        db.collection("secondhand").add({
          data: {
            big_class: "实习",
            Date: that.postDate,
            Time: that.postTime,
            jobName: that.jobName,
            field: that.field,
            workUnit: that.workUnit,
            workPlace: that.workPlace,
            jobDescription: that.jobDescription,
            salary: that.salary,
            resumeSend: that.resumeSend,
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