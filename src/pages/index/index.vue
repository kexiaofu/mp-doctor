<template>
  <div class="container" @click="clickHandle('test click', $event)">

    <div class="userinfo" @click="bindViewTap">
      <img class="userinfo-avatar" v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl" background-size="cover" />
      <div class="userinfo-nickname">
        <p>{{userInfo.nickName}}</p>
      </div>
    </div>

    <button open-type="getUserInfo" @getuserinfo="hadGetInfo" v-show="noAuthority">请登录</button>

  </div>
</template>

<script>
import { apiUrl } from '@/utils/index'

export default {
  data () {
    return {
      userInfo: {},
      noAuthority:false,
      openid:''
    }
  },

  components: {},

  mounted() {
    console.log(apiUrl)
  },

  methods: {
    bindViewTap () {
      const url = '../logs/main'
      wx.navigateTo({ url })
    },
    getUserInfo () {
      let oid = wx.getStorageSync('oid')
      console.log(oid === '');

      if(oid) {
        wx.checkSession({
          success:res=>{
            console.log('session 未过期')
            this.queryUser(oid);
          },
          fail:res=>{
            console.log('session 过期了')
            this.wxLogin();
          }
        })
      } else {
        console.log('需要登录');
        this.wxLogin();
      }

    },

    wxLogin () {
      wx.login({
        success:res=>{
          console.log('拿到code了')
          this.queryLogin(res.code);
        }
      })
    },

    queryLogin (code) {
      wx.request({
        url: apiUrl + '/api/getUserInfo',
        data:{
          code:code
        },
        success:res=>{
          console.log(res);
          if(res.data.code === 2000) {
            let data = res.data.data;
            if(data.hadRecord) {
              console.log('老用户');
              this.userInfo = data.record[0];
              wx.setStorageSync('oid',data.clientId);
            } else {
              this.noAuthority = true;
              console.log('新用户',this.noAuthority);
            }
            this.openid = data.clientId
          }
        }
      })
    },

    hadGetInfo (e) {
      this.noAuthority = false;
      this.userInfo = e.mp.detail.userInfo;
      this.userInfo.openid = this.openid;
      console.log('拿到了身份了');
      wx.setStorageSync('oid',this.openid);
      wx.setStorageSync('userInfo',JSON.stringify(this.userInfo));
      wx.request({
        url:apiUrl + '/api/setUserInfo',
        method:'POST',
        data:this.userInfo
      })
    },

    queryUser (id) {
      wx.request({
        url:apiUrl + '/api/queryUser',
        data:{id},
        success:res=>{
          if(res.data.code === 2000) {
            this.userInfo = res.data.data[0];
          } else if(res.data.code === 4004) {
             this.wxLogin();
          }
        },
      })
    },
    clickHandle (msg, ev) {
      console.log('clickHandle:', msg, ev)
    }
  },

  created () {
    // 调用应用实例的方法获取全局数据
    this.getUserInfo()
  }
}
</script>

<style scoped>
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

</style>
