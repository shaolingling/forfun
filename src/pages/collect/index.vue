<template>
  <div class="container">
    <div class="tab_top">
       <wux-icon type="md-create"  size="32"  class="icon_create"/>
      
       <input  class="input_val"  v-model.lazy="newVal"  @change="addItem" placeholder="接下去要做什么？"/> 
         <ul class="content">
             <li v-for = "(innerItem ,index) in toDoList" :key ="index">
                              <div class="check_box" :class="innerItem.done ?'checked':''"   @click="innerItem.done =!innerItem.done" v-if="!innerItem.done"></div>
                               <wux-icon type="ios-checkmark-circle"  @click="innerItem.done =!innerItem.done" v-else/>
                              <a  href="/pages/task/main?id=index"  :class="innerItem.done ?'line_through':''"          
                               >{{innerItem.val}}</a>
             </li>                
        </ul>
    </div>
  </div>
</template>

<script>
// import card from '@/components/card'

import Vue from "vue";
export default {
  data() {
    return {
      collection:'',//数据库表名称，如工作，电影
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
      ]
    };
  },

  methods: {
    addItem() {
      if(!this.newVal.trim()) return;
      this.toDoList.push({ val: this.newVal, done: false });
      this.collection.add({
        // data 字段表示需新增的 JSON 数据
        data: {
          timeStamp: Date.now(),
          val: this.newVal, 
          done: false
        },
        success: res => {
          // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
          console.log(res);
        },
      });
      this.newVal = "";
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
  
  
  },

  created() {
    // 调用应用实例的方法获取全局数据
    this.getUserInfo();
  },
  onLoad: function(option){
    console.log(option.id)
    wx.cloud.init({
      env: "forfun-3ed578",
      traceUser: true
    });
    const forFunDB = wx.cloud.database({
      env: "forfun-3ed578"
    });
    this.collection = forFunDB.collection(option.id);
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
    word-break: break-all;
    .check_box {
      width: 50rpx;
      height: 50rpx;
      border: 1rpx solid #ccc;
      margin-right: 20rpx;
      border-radius: 50%;
      flex-shrink: 0;
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

