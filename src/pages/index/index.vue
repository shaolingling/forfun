<template>
  <div class="container">
       <!-- <a href="/pages/collect/main?id=work" class="counter">工作</a>  
       <a href="/pages/collect/main?id=movie" class="counter">电影</a>  -->
       <ul>
          <li v-for = "(item,index) in collectionList" :key = "index"  @click="toList(item.packageName)"
          > 
          {{item.packageName}}
          </li>
       </ul>
      
       <div @click="showInput = true">+创建分类</div>
       <input  v-if="showInput" class="input_val"  v-model.lazy="newVal"  @change="addPackage" placeholder="新增分类"/> 
  </div>
</template>

<script>
// import card from '@/components/card'

import Vue from "vue";
export default {
  data() {
    return {
      showInput: false,
      newVal: "",
      index: 0,
      key: "tab1",
      current: "tab1",
      collectionList: [],
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
      forFunDB: ""
    };
  },

  methods: {
    toList(packageName) {
      wx.navigateTo({
        url: "../collect/main?packageName=" + packageName
      });
    },
    addPackage() {
      this.showInput = false;
      this.forFunDB.collection("collection").add({
        // data 字段表示需新增的 JSON 数据
        data: {
         // timeStamp: Date.now(),
          packageName: this.newVal,
        },
        success: res => {
          // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
          console.log(res);
          this.collectionList.push({packageName: this.newVal});
          this.newVal = "";
        }
      });
    },
    bindViewTap() {
      const url = "../logs/main";
      wx.navigateTo({ url });
    },
    getUserInfo() {
      // 调用登录接口
      wx.login({
        success: () => {
          wx.getUserInfo({
            success: res => {
              this.userInfo = res.userInfo;
            }
          });
        }
      });
    },
    onTabsChange(e) {
      console.log("onTabsChange", e);
      const { key } = e.mp.detail;
      const index = this.tabs.map(n => n.key).indexOf(key);
      this.key = key;
      this.index = index;
      console.log(this.key);
    }
    // onSwiperChange(e) {
    //   console.log("onSwiperChange", e);
    //   const { current: index, source } = e.mp.detail;
    //   const { key } = this.tabs[index];
    //   if (!!source) {
    //     this.key = key;
    //     this.index = index;
    //   }
    // }
  },

  created() {
    // 调用应用实例的方法获取全局数据
    this.getUserInfo();
  },
  onLoad: function() {
    wx.cloud.init({
      env: "forfun-3ed578",
      traceUser: true
    });
     this.forFunDB = wx.cloud.database({
      env: "forfun-3ed578"
    });
    this.collection = this.forFunDB.collection("collection");
    this.collection.get({
      success: res => {
        // res.data 是一个包含集合中有权限访问的所有记录的数据，不超过 20 条
        console.log(res.data);
        this.collectionList = res.data
      }
    });
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
}
</style>
