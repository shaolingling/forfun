<template>
  <div class="collection_box">
      <!-- <wux-upload url="https://www.skyvow.cn/api/common/file" @change="onChange" @success="onSuccess" @fail="onFail" @complete="onComplete"> -->
         <div>+添加备注</div>
         <textarea   @blur = "addRemark" v-model="remark"></textarea>
         <div class="add_item"  @click="chooseImage">+添加照片</div>
         <ul>
           <li v-for="(item,index) in imgInfos" :key="index" class="image_item">
              <image :src="item.src" @click="previewImage(item.src)"/>   
              <wux-progress  v-if= "item.progress<100"  bar-style="border:1px solid #ccc"   active-color="#ddd"   :percent="item.progress"  class = "progress_item" />  
              <!-- <div v-else>已经上传</div>          -->
           </li>
         </ul>
          <button @click="voiceToggle" class='btn'>录音按钮</button>
          <div v-if="notStartVoice">点击按钮，开始录音</div>
          <div v-else>正在录音中...,点击按钮，停止录音</div>
          <div>录音上传中...</div>
          <div>上传完毕</div>

          <!-- <button @click="stopVoice" class='btn'>停止录音</button> -->
          <!-- <button @click="playVoice" class='btn'>播放录音</button> -->
      <!-- </wux-upload> -->
  </div>
     
 
    <!-- <div class="userinfo" @click="bindViewTap">
      <img class="userinfo-avatar" v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl" background-size="cover" />
      <div class="userinfo-nickname">
        <card :text="userInfo.nickName"></card>
      </div>
    </div>

    <div class="usermotto">
      <div class="user-motto">
        <card :text="motto"></card>
      </div>
    </div>

    <form class="form-container">
      <input type="text" class="form-control" v-model="motto" placeholder="v-model" />
      <input type="text" class="form-control" v-model.lazy="motto" placeholder="v-model.lazy" />
    </form>
    <a href="/pages/counter/main" class="counter">去往Vuex示例页面</a> -->
  
</template>

<script>
// import card from '@/components/card'
import Vue from "vue";
export default {
  data() {
    return {
      tempVoicePath: "",
      notStartVoice: true,
      recorderManager: null,
      imgInfos: [],
      // imageUrls: [],
      imgCount: 0,
      newVal: "",
      index: 0,
      key: "tab1",
      current: "tab1",
      toDoList: [],
      tabs: [
        {
          key: "tab1",
          title: "所有事情",
          lists: []
        },
        {
          key: "tab2",
          title: "未完成事情",
          lists: []
        },
        {
          key: "tab3",
          title: "已完成事情",
          lists: []
        }
      ],
      works: "", //云平台数据库里的集合的引用
      taskId: "",
      forFunDB: null,
      remark: ""
    };
  },
  onLoad: function(option) {
    console.log(option);
    console.log(option.id);
    console.log(option.collectionId);
    this.taskId = option.id;
    wx.cloud.init({
      env: "forfun-3ed578",
      traceUser: true
    });
    const forFunDB = wx.cloud.database({
      env: "forfun-3ed578"
    });
    this.forFunDB = forFunDB;
    this.collection = forFunDB.collection(option.collectionId);
    this.collection.doc(option.id).get({
      success: res => {
        // res.data 是一个包含集合中有权限访问的所有记录的数据，不超过 20 条
        console.log(res.data);
        if (res.data) {
          this.remark = res.data.remark;
          let photosIdArr = res.data.photos || [];
          this.getTempFileURL(photosIdArr);
        }
      }
    });
    this.recorderManager = wx.getRecorderManager();
    this.innerAudioContext = wx.createInnerAudioContext();
  },
  methods: {
    chooseImage() {
      //这里是选取图片的方法
      wx.chooseImage({
        //count: 9 - pics.length, // 最多可以选择的图片张数，默认9
        sizeType: ["original", "compressed"], // original 原图，compressed 压缩图，默认二者都有
        sourceType: ["album", "camera"], // album 从相册选图，camera 使用相机，默认二者都有
        success: res => {
          let imgsrc = res.tempFilePaths;
          let imgInfosAdd = imgsrc.map(item => ({
            src: item,
            progress: 0
          }));
          //   this.imageUrls.unshift(...imgsrc);
          this.imgInfos.unshift(...imgInfosAdd);
          //这里触发图片上传的方法
          this.uploadimg({
            path: imgInfosAdd //这里是选取的图片的地址数组和上传进度，不止一张图片
          });
        },
        fail: function() {
          // fail
        },
        complete: function() {
          // complete
        }
      });
    },
    uploadFile(tempPath) {
      wx.cloud.uploadFile({
        // url: data.url,
        filePath: tempPath,
        cloudPath: "collection/voice/" + Math.random(),
        // name: "file", //这里根据自己的实际情况改
        // formData: null, //这里是上传图片时一起上传的数据
        success: resp => {
          //console.log(uploadTask.onProgressUpdate())
          //success++; //图片上传成功，图片上传成功的变量+1
          console.log(resp);
          console.log("上传成功");
         // this.addTaskDetail(undefined, resp.fileID);
          //这里可能有BUG，失败也会执行这里,所以这里应该是后台返回过来的状态码为成功时，这里的success才+1
        },
        fail: res => {
          console.log(res)          
        },
        complete: res => {
          console.log(res)
        }
      });
    },
    voiceToggle() {
      this.notStartVoice ? this.startVoice() : this.stopVoice();
      this.notStartVoice = !this.notStartVoice;
    },
    startVoice() {
      const options = {
        //  duration: 10000, //指定录音的时长，单位 ms
        sampleRate: 16000, //采样率
        numberOfChannels: 1, //录音通道数
        encodeBitRate: 96000, //编码码率
        format: "mp3", //音频格式，有效值 aac/mp3
        frameSize: 50 //指定帧大小，单位 KB
      };
      //开始录音
      this.recorderManager.start(options);
      this.recorderManager.onStart(() => {
        console.log("recorder start");
      });
      //错误回调
      this.recorderManager.onError(res => {
        console.log(res);
      });
    },
    //停止录音
    stopVoice() {
      this.recorderManager.stop();
      this.recorderManager.onStop(res => {
        this.tempVoicePath = res.tempFilePath;
        console.log("停止录音", res.tempFilePath);
        this.uploadFile(this.tempVoicePath) 
        //  const { tempFilePath } = res;
      });
    },
    playVoice() {
      this.innerAudioContext = wx.createInnerAudioContext();
      this.innerAudioContext.onError(res => {
        // 播放音频失败的回调
      });
      this.innerAudioContext.src = this.tempVoicePath; // 这里可以是录音的临时路径
      this.innerAudioContext.play();
    },
    //多张图片上传
    uploadimg(data) {
      let that = this,
        i = data.i ? data.i : 0, //当前上传的哪张图片
        success = data.success ? data.success : 0, //上传成功的个数
        fail = data.fail ? data.fail : 0; //上传失败的个数
      let uploadTask = wx.cloud.uploadFile({
        // url: data.url,
        filePath: data.path[i].src,
        cloudPath: "collection/imgs/" + Math.random(),
        // name: "file", //这里根据自己的实际情况改
        // formData: null, //这里是上传图片时一起上传的数据
        success: resp => {
          //console.log(uploadTask.onProgressUpdate())
          success++; //图片上传成功，图片上传成功的变量+1
          console.log(resp);
          console.log("上传成功");
          this.addTaskDetail(undefined, resp.fileID);
          //这里可能有BUG，失败也会执行这里,所以这里应该是后台返回过来的状态码为成功时，这里的success才+1
        },
        fail: res => {
          fail++; //图片上传失败，图片上传失败的变量+1
          console.log("fail:" + i + "fail:" + fail);
        },
        complete: () => {
          console.log(i);
          i++; //这个图片执行完上传后，开始上传下一张
          if (i == data.path.length) {
            //当图片传完时，停止调用
            console.log("执行完毕");
            console.log("成功：" + success + " 失败：" + fail);
          } else {
            //若图片还没有传完，则继续调用函数
            console.log(i);
            data.i = i;
            data.success = success;
            data.fail = fail;
            that.uploadimg(data);
          }
        }
      });
      uploadTask.onProgressUpdate(res => {
        console.log("上传进度", res.progress);
        data.path[i].progress = res.progress;
        console.log("已经上传的数据长度", res.totalBytesSent);
        console.log("预期需要上传的数据总长度", res.totalBytesExpectedToSend);
      });
    },
    getTempFileURL(fileIDs) {
      wx.cloud.getTempFileURL({
        fileList: fileIDs,
        success: res => {
          // get temp file URL
          console.log(res.fileList);
          this.imgInfos = res.fileList.map(item => ({
            src: item.tempFileURL,
            progress: 100
          }));
        },
        fail: err => {
          console.log(err);
        }
      });
    },
    //将文件添加到数据库
    addTaskDetail(remark = "", photoId = "") {
      console.log(remark);
      console.log(this.taskId);
      const _ = this.forFunDB.command;
      let data = null;
      if (remark && !photoId) {
        data = {
          remark: remark
        };
      }
      if (!remark && photoId) {
        data = {
          photos: _.push(photoId)
        };
      }
      if (remark && photoId) {
        data = {
          remark: remark,
          photos: _.push(photoId)
        };
      }
      this.collection.doc(this.taskId).update({
        // data 传入需要局部更新的数据
        data: data,
        success: function(res) {
          console.log(res.data);
        },
        fail: function(res) {
          console.log(res);
        }
      });
    },
    previewImage(url) {
      wx.previewImage({
        current: url, // 当前显示图片的http链接
        urls: [url] // 需要预览的图片http链接列表
      });
    },
    addRemark() {
      // console.log(event.mp.detail.value);
      // let remark = event.mp.detail.value
      this.addTaskDetail(this.remark);
    }
  }
};
</script>

<style scoped lang="scss">
.collection_box {
  .image_item {
    height: 200rpx;
    overflow: hidden;
    text-align: center;
    position: relative;
    padding: 20rpx 0;
    width: 80%;
    image {
      width: 100%;
      border-radius: 6rpx;
    }
    .progress_item {
      position: absolute;
      bottom: 20rpx;
      left: 10%;
      width: 80%;
    }
  }
}
</style>
