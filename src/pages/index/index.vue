<template>
  <div class="container">
       <a href="/pages/todolist/main" class="counter">去往记事本</a>  
       <a href="/pages/collect/main" class="counter">去往收藏夹</a>  
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
      newVal: "",
      index: 0,
      key: "tab1",
      current: "tab1",
      toDoList:[],
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
      ]
    };
  },

  methods: {
    
    addItem() {
      this.toDoList.push({ val: this.newVal, done: false });
      this.newVal = "";
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
    },
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
    .ios-checkmark-circle{
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
