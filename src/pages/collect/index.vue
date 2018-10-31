<template>
  <div class="container">
       收藏夹
      <!-- <wux-upload url="https://www.skyvow.cn/api/common/file" @change="onChange" @success="onSuccess" @fail="onFail" @complete="onComplete"> -->
         <div class="add_item"  @click="upLoadImg">+添加照片</div>
         <ul>
           <li v-for="(item,index) in imageUrls" :key="index">
               <image :src="item"/>               
           </li>
         </ul>
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
      imageUrls: [],
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
      photos: "" //云平台数据库里的集合的引用
    };
  },
  async onReady() {
    wx.cloud.init({
      env: "forfun-3ed578"
    });
    const forFunDB = wx.cloud.database({
      env: "forfun-3ed578"
    });
    this.photos = forFunDB.collection("photo");
    await this.photos.get({
      success: res  => {
        // res.data 是一个包含集合中有权限访问的所有记录的数据，不超过 20 条
        console.log(res.data);
        let fileIdArr = res.data.map(item => item.fileID)
        this.imgCount = fileIdArr.length
        console.log(fileIdArr)
        wx.cloud.getTempFileURL({
                fileList: fileIdArr,
                success: res => {
                  // get temp file URL
                  console.log(res.fileList);
                  this.imageUrls = res.fileList.map(item => item.tempFileURL)
                },
                fail: err => {
                  console.log(err);
                }
              });
      }
    });
    
   // let _this = this;
    // wx.getStorage({
    //   key: "imageUrls",
    //   success(res) {
    //     console.log(res.data);
    //     _this.imageUrls = JSON.parse(res.data);
    //   }
    // });
  },
  methods: {
   async  upLoadImg() {
      // 让用户选择一张图片
      wx.chooseImage({
        success: chooseResult => {
          // 将图片上传至云存储空间
          wx.cloud.uploadFile({
            // 指定上传到的云路径
            cloudPath: "collection/imgs/" + this.imgCount,
            // 指定要上传的文件的小程序临时文件路径
            filePath: chooseResult.tempFilePaths[0],
            // 成功回调
            success: res => {
              console.log("上传成功", res);
              this.imgCount++;
      
            wx.cloud.getTempFileURL({
                fileList: [res.fileID],
                success: res => {
                  // get temp file URL
                  console.log(res.fileList);
                  this.imageUrls.unshift(res.fileList[0].tempFileURL);
                  
                  // wx.setStorage({
                  //   key: "imageUrls",
                  //   data: JSON.stringify(this.imageUrls)
                  // });
                },
                fail: err => {
                  console.log(err);
                }
              });
              this.photos.add({
                // data 字段表示需新增的 JSON 数据
                data: {
                  fileID: res.fileID
                },
                success: res => {
                  // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
                  console.log(res);
                }
              });
            }
          });
        }
      });
    }
    // onChange(e) {
    //     console.log('onChange', e)
    //     const { file } = e.mp.detail
    //     if (file.status === 'uploading') {
    //         this.progress = 0,
    //         wx.showLoading()
    //     } else if (file.status === 'done') {
    //        this.imageUrl = file.url
    //     }
    // },
    // onSuccess(e) {
    //     console.log('onSuccess', e)
    // },
    // onFail(e) {
    //     console.log('onFail', e)
    // },
    // onComplete(e) {
    //     console.log('onComplete', e)
    //     wx.hideLoading()
    // },
  }
};
</script>

<style scoped lang="scss">
.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.userinfo-avatar {
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.userinfo-nickname {
  color: #aaa;
}

.usermotto {
  margin-top: 150px;
}

.form-control {
  display: block;
  padding: 0 12px;
  margin-bottom: 5px;
  border: 1px solid #ccc;
}

.counter {
  display: inline-block;
  margin: 10px auto;
  padding: 5px 10px;
  color: blue;
  border: 1px solid blue;
}
.container {
  width: 100vw;
  padding: 0 6vw;
  .content li {
    display: flex;
    .check_box {
      width: 50rpx;
      height: 50rpx;
      border: 1rpx solid #ccc;
      margin-right: 20rpx;
      border-radius: 50%;
    }
    .ios-checkmark-circle {
      margin-right: 20rpx;
    }
  }

  .tab_top {
    width: 100%;
  }
  swiper {
    height: 80vh;
  }
  ._wux-tabs {
    position: fixed;
    bottom: 0;
    .wux-tabs {
      display: flex;
    }
  }
  ._wux-tab {
    width: 220rpx;
  }
  .wux-tabs__tab--current {
    background-color: #33cd5f;
    color: #fff;
  }
  .input_val {
    height: 6vh;
    border: 1px solid #eee;
    border-radius: 3vh;
    padding-left: 10vw;
  }
  .icon_create {
    position: relative;
    top: 6vh;
    left: 3vw;
  }
  .line_through {
    text-decoration: line-through;
  }
  .add_item {
    width: 200rpx;
    line-height: 200rpx;
    border: 1px solid #000;
  }
}
</style>
