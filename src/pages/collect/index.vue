<template>
  <div class="container">
    <div class="tab_top">
       <wux-icon type="md-create"  size="32"  class="icon_create"/>
      
       <input  class="input_val"  v-model.lazy="newVal"  @change="addItem" placeholder="接下去要做什么？"/> 
         <ul class="content">
             <li v-for = "(item ,index) in toDoList" :key ="index">
                   <div class="check_box"   @click="doneChange(item)"></div>
                   <div  @click="toTaskDetail(item._id,collectionId)">{{item.task}}</div>
             </li>                
        </ul>
        <div v-if="hideDone" @click="hideDone = false">显示已完成的任务</div>
        <div v-else @click="hideDone = true">隐藏已完成的任务</div>
        <div></div>
        <ul class="content" v-if="!hideDone">
             <li v-for = "(item ,index) in doneList" :key ="index">
                    <wux-icon type="ios-checkmark-circle"  @click="doneChange(item)" />
                     <div  @click="toTaskDetail(item._id,collectionId)"  :class=" line_through">{{item.val}}</div>
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
      forFunDB:'',
      collection: "", //数据库表名称，如工作，电影
      taskcollection:'',
      taskList:"",
      packageName: "",
      newVal: "",
      index: 0,
      key: "tab1",
      current: "tab1",
      toDoList: [],
      doneList: [],
      hideDone:true,
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
    toTaskDetail(id, collectionId) {
      console.log(id, collectionId);
      wx.navigateTo({
        url: "../task/main?id=" + id + "&collectionId=" + collectionId
      });
    },
    addItem() {
       this.taskcollection.add({
        // data 字段表示需新增的 JSON 数据
        data: {
          packageName: this.packageName,
          task:this.newVal,
          done:false
        },
        success: res => {
           console.log(res.data,"插入成功");
           this.toDoList.push({"task":this.newVal,"done":false})
           this.newVal = "";
        }
      });
    },
    doneChange(item) {
      const _ = this.forFunDB.command;
      let data={}
      this.taskcollection.doc(item.task).update({
        // data 传入需要局部更新的数据
        data: {
          done: !item.done
        },
        success: res => {
          console.log(item.done,"更新成功")
          // item.done
          //   ? this.doneList.push(item) &&
          //     this.toDoList.splice(this.toDoList.indexOf(item), 1)
          //   : this.toDoList.push(item) &&
          //     this.doneList.splice(this.doneList.indexOf(item), 1);
        },
        fail:res => {
          console.log(res)
        }
      });
    },
    // async addItem() {
    //   if (!this.newVal.trim()) return;
    //   await this.collection.add({
    //     // data 字段表示需新增的 JSON 数据
    //     data: {
    //       timeStamp: Date.now(),
    //       val: this.newVal,
    //       done: false,
    //       remark: "",
    //       photos: [],
    //       voice:""
    //     },
    //     success: res => {
    //       // res 是一个对象，其中有 _id 字段标记刚创建的记录的 id
    //       console.log(res);
    //       this.toDoList.push({ val: this.newVal, done: false });
    //       this.newVal = "";
    //     }
    //   });
    //   // await this.collection.get({
    //   //   success: res => {
    //   //     // res.data 是一个包含集合中有权限访问的所有记录的数据，不超过 20 条
    //   //     console.log(res.data);
    //   //     this.toDoList = res.data;
    //   //   }
    //   // });
    // },

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
    }
  },

  created() {
    // 调用应用实例的方法获取全局数据
    this.getUserInfo();
  },
  onLoad: function(option) {
    console.log(option.packageName);
    this.packageName = option.packageName;
    wx.cloud.init({
      env: "forfun-3ed578",
      traceUser: true
    });
     this.forFunDB = wx.cloud.database({
      env: "forfun-3ed578"
    });
    const _ = this.forFunDB.command;
    this.taskcollection = this.forFunDB.collection("taskList")
    this.taskcollection.where({
       packageName: _.eq(this.packageName)
    }).get({
      success: res => {
        // res.data 是一个包含集合中有权限访问的所有记录的数据，不超过 20 条
        console.log(res.data,"taskList");
        let taskList = res.data 
        this.toDoList = taskList.filter(item => !item.done);
        this.doneList = taskList.filter(item => item.done);
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

