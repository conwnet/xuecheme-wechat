<template>
  <div>
    <mt-navbar v-model="page" fixed>
      <mt-tab-item id="enroll">套餐详情</mt-tab-item>
      <mt-tab-item id="comment">学员评价</mt-tab-item>
      <mt-tab-item id="chat">在线客服</mt-tab-item>
    </mt-navbar>

    <mt-tab-container id="container" v-model="page">
      <mt-tab-container-item id="enroll" class="enroll-page">
         <mt-swipe class="banner" :auto="4000">
            <mt-swipe-item v-for="(item, index) in banners" :key="'banner-' + index"><img :src="item.src" /></mt-swipe-item>
          </mt-swipe>
          <div class="panel">
            <div class="info">
              <div class="logo"><img :src="logo.src" /></div>
              <div class="content">
                <h1>{{ this.title }}</h1>
                <p class="price">
                  <span>¥</span>
                  <span style="font-size: 30px;">{{ parseInt(pack.price / 100) || '???' }}</span>
                </p>
              </div>
            </div>
            
            
            <div style="display: flex; color: #888; font-size: 12px; margin-top: 10px;">
              <div style="flex: 1; text-align:center;">已报名：{{ order_cnt }}</div>
              <div style="flex: 1; text-align:center;">评价：{{ comment_cnt }}</div>
              <div style="flex: 1; text-align:center;">辽宁沈阳</div>
            </div>
          </div>

          <mt-cell class="part" title="已选套餐" is-link @click.native="packSheetVisible = !packSheetVisible">
            <mt-spinner type="double-bounce" color="#26a2ff" v-if="!pack.name"></mt-spinner>
            {{ pack.name }}
          </mt-cell>

          <div class="part panel">
            <mt-spinner type="triple-bounce" color="#26a2ff" v-if="!pack.name"></mt-spinner>
            {{ pack.desc }}
          </div>

          <div class="part">
            <mt-button style="width: 95%; margin: 0 auto;" type="primary" size="large" @click="enrollPopupVisible = true">立即报名</mt-button>
          </div>

          <mt-popup v-model="enrollPopupVisible" position="right" class="confimPage" :modal="false">
            <mt-header fixed title="报名信息">
              <div slot="left">
                <mt-button icon="back" @click="enrollPopupVisible = false">返回</mt-button>
              </div>
            </mt-header>

            <div style="margin-top: 40px;">
              <mt-field label="微信" placeholder="获取失败，请重新进入" v-model="nickname" :state="checkFormat('nickname') ? 'success' : 'error'" readonly disableClear></mt-field>
              <mt-field label="姓名" placeholder="怎么称呼您？" :state="checkFormat('name') ? 'success' : 'error'" v-model="name"></mt-field>
              <mt-field label="手机号" placeholder="怎么联系您？" :state="checkFormat('mobile') ? 'success' : 'error'" v-model="mobile"></mt-field>
              <mt-field label="验证码" placeholder="请查收短信～" :state="checkFormat('verify') ? 'success' : 'error'" v-model="verify">
                <mt-button type="primary" @click="sendVerify()" :disabled="verifyButton.disabled">{{ verifyButton.text }}</mt-button>
              </mt-field>
              <mt-field label="优惠码" placeholder="可在公众号内获取" v-model="promo"></mt-field>
            </div>
            <div class="part">
              <mt-button style="width: 95%; margin: 0 auto;" type="primary" size="large" @click="confirmEnroll">微信支付</mt-button>
            </div>
          </mt-popup>

          <mt-actionsheet :actions="packNames" v-model="packSheetVisible">
          </mt-actionsheet>
      </mt-tab-container-item>

      <mt-tab-container-item id="comment" class="comment-page">
        <div class="nav">
          <mt-button size="small" class="nav-btn" :icon="alreadyEnroll ? 'more' : ''" @click="alreadyEnroll = !alreadyEnroll">已报名</mt-button>
          <mt-button size="small" class="nav-btn" :icon="notEnroll ? 'more' : ''" @click="notEnroll = !notEnroll">未报名</mt-button>
          <span style="flex: 1"></span>
          <mt-button size="small" type="primary" class="nav-btn" @click="inputBox = !inputBox" :icon="inputBox ? 'more' : ''">我也说说</mt-button>
        </div>

        <transition name="slide-fade">
          <div class="input" v-if="inputBox">
            <mt-field type="textarea" rows="3" style="border: 1px solid #bbb;" v-model="comment"></mt-field>
            <mt-button size="small" type="primary" class="fb-btn" @click="addComment">发表</mt-button>
            <mt-button size="small" class="fb-btn" @click="inputBox = !inputBox">取消</mt-button>
          </div>
        </transition>

        <mt-spinner type="snake" color="#26a2ff" v-if="!comments.length" :size="40" style="display: block; width: 40px; margin: 0 auto; padding: 10px;"></mt-spinner>

        <transition-group name="slide-fade">
          <div class="comment" v-for="(item, index) in comments" :key="'comment-' + index" v-if="showComment(item.status)">
            <img class="comment-headimg" :src="item.headimgurl" />
            <div class="comment-arrow"></div>
            <div class="comment-container">
              <div class="comment-title">
                <span class="comment-nickname">{{ item.nickname }}</span>
                <span>{{ item.date }}</span>
                <span>({{ item.status ? '已报名' : '未报名' }})</span>
              </div>
              <div class="comment-content">
                <p>{{ item.content }}</p>
              </div>
            </div>
          </div> 
        </transition-group>


      </mt-tab-container-item>

      <mt-tab-container-item id="chat">
        <p style="text-align: center;">即将上线！</p>
      </mt-tab-container-item>

    </mt-tab-container>

    <mt-popup v-model="messagePopupVisible" class="messagePopup" position="top" :modal="false">
      {{ message }}
    </mt-popup>

  </div>
</template>

<script>

import axios from 'axios'
import config from './config'
import wx from 'weixin-js-sdk'

let _get = (url, params, callback) => {
  let ssid = localStorage.ssid
  let timeout = parseInt(localStorage.timeout)

  if (!ssid || timeout - 600000 < Date.now()) {
    console.log('身份认证失败，请重新进入...')
  } else {
    axios.get(url, { params: params, headers: { ssid: ssid } }).then(callback)
  }
}

let _post = (url, data, callback) => {
  let ssid = localStorage.ssid
  let timeout = parseInt(localStorage.timeout)

  if (!ssid || timeout - 600000 < Date.now()) {
    console.log('身份认证失败，请重新进入...')
  } else {
    axios.post(url, data, { headers: { ssid: ssid } }).then(callback)
  }
}

export default {
  name: 'app',
  data () {
    return {
      page: '', // NavBar 的选项值
      messagePopupVisible: false,
      // enroll page
      packSheetVisible: false,
      enrollPopupVisible: false,
      order_cnt: 0,
      comment_cnt: 0,
      message: '',
      nickname: '',
      name: '',
      title: '',
      mobile: '',
      verify: '',
      promo: '',
      pack: {},
      logo: {},
      packs: [],
      banners: [],
      verifyButton: { disabled: false, text: '立即发送' },
      // comment page
      comments: [],
      comment: '',
      inputBox: false,
      alreadyEnroll: true,
      notEnroll: true
    }
  },
  computed: {
    packNames () {
      let ret = []
      this.packs.map(item => {
        ret.push({ name: item.name, method: () => { this.pack = item } })
      })
      return ret
    }
  },
  watch: {
    page (val) {
      if (val === 'enroll') {
        _get (config.apiUrl + 'school', {}, res => {
          if (res.status === 200) {
            let data = res.data
            this.order_cnt = data.order_cnt
            this.comment_cnt = data.comment_cnt
            this.nickname = data.nickname
            this.banners = data.banners
            this.title = data.title
            this.logo = data.logo
            this.packs = data.packs
            this.pack = this.packs[0]
          }
        })
      } else if (val === 'comment') {
        _get (config.apiUrl + 'comments', {}, res => {
          if (res.status === 200) {
            this.comments = res.data
          }
        })
      }
    }
  },
  methods: {
    Message (message) {
      this.messagePopupVisible = true
      this.message = message
      setTimeout(() => {
        this.messagePopupVisible = false
        this.message = ''
      }, 2000)
    },
    sendVerify (message) {
      if (this.checkFormat('mobile')) {
        _get(config.apiUrl + 'verify', { mobile: this.mobile }, res => {
          if (res.status === 200) {
            this.Message(res.data.errmsg)
            let second = 60
            this.verifyButton = { disabled: true, text: '60s' }
            let intval = setInterval(() => {
              this.verifyButton = { disabled: true, text: --second + 's' }
            }, 1000)
            setTimeout(() => {
              clearInterval(intval)
              this.verifyButton = { disabled: false, text: '再次发送' }
            }, 60000)
          } else {
            this.Message('网络貌似开小差了...')
          }
        })
      } else {
        this.Message('手机号格式不正确哦，这是中国号码吗？')
      }
    },
    showComment (status) {
      return (status && this.alreadyEnroll) || (!status && this.notEnroll)
    },
    checkFormat (type) {
      if (type === 'name') {
        return this.name !== ''
      } else if (type === 'mobile') {
        return /1[0-9]{10}/.test(this.mobile) && this.mobile.length === 11
      } else if (type === 'verify') {
        return /[0-9]+/.test(this.verify)
      } else if (type === 'nickname') {
        return this.nickname !== ''
      }
    },
    addComment () {
      _post(config.apiUrl + 'comment', { content: this.comment }, res => {
        if (res.status === 200) {
          this.Message(res.data.errmsg)
          if (!res.data.errcode) this.inputBox = !this.inputBox
        }
      })
    },
    confirmEnroll () {
      if (!this.checkFormat('name')) this.Message('请问如何称呼您？')
      else if (!this.checkFormat('mobile')) this.Message('请问如何联系您？')
      else if (!this.checkFormat('verify')) this.Message('我们需要验证码以确定您没有错误输入了手机号')
      else {
        _post(config.apiUrl + 'order', {
          name: this.name, verify: this.verify, promo: this.promo, packId: this.pack.id
        }, res => {
          if (res.status === 200) {
            let data = res.data
            if (data.errcode) {
              this.Message(data.errmsg)
            } else {
              _get (config.apiUrl + 'payParams', {}, res => {
                if (res.status === 200) {
                  let data = res.data
                  wx.chooseWXPay({
                    timestamp: data.timeStamp,
                    nonceStr: data.nonceStr,
                    package: data.package,
                    signType: data.signType,
                    paySign: data.paySign,
                    success: res => {
                      this.Message('支付成功！');
                    }
                  })
                }
              })
            }
          }
        })
      }
    }
  },
  created () {
    this.page = 'enroll'
    _get (config.apiUrl + 'jsSdkConfig', { url: location.href.split('#')[0] }, res => {
      if (res.status === 200) {
        wx.config(res.data);
      }
    })
  }
}
</script>

<style lang="scss" type="text/css">
  
  body {
    margin: 0;
    background-color: #f4f4f4;
  }

  #container {
    margin-top: 49px;
  }

  .messagePopup {
    width: 100%;
    height: 50px;
    text-align: center;
    background-color: rgba(0,0,0,.7);
    backface-visibility: hidden;
    line-height: 50px;
    color: #fff;
  }

  // enroll page
  .enroll-page {

    .banner {
      width: 100%;
      height: 200px;

      img {
        width: 100%;
        height: 100%;
      }
    }

    .part {
      margin-top: 10px;
    }

    .panel {
      padding: 5px;
      background-color: #fff;

      .info {
        display: flex;

        .logo {
          width: 80px;
          height: 80px;
        }

        .content {
          flex: 1;
          margin-left: 10px;

          h1 {
            font-size: 18px;
            margin: 05px 0;
            font-weight: normal;
          }

          .price {
            flex: 1;
            color: #eb5211;
            font-size: 22px;
            margin: 0;
          }
        }
      }
    }

    .confimPage {
      width: 100%;
      height: 100%;
    }

  }

  /* comments page */

  .comment-page {

    .nav {
      display: flex;
      padding: 5px;

      .nav-btn {
        margin: 0 5px;
      }
    }

    .input {
      margin: 5px 10px;

      .fb-btn {
        margin-top: 5px;
      }
    }

    .comment {
      display: flex;
      margin: 5px 10px;

      .comment-headimg {
        width: 50px;
        height: 50px;
        border-radius: 3px; 
        display: block;
      }

      .comment-arrow {
        height: 10px;
        width: 10px;
        background-color: #eee;
        border-left: 1px solid #bbb;
        border-bottom: 1px solid #bbb;
        position: relative;
        left: 6px;
        top: 15px; 
        transform :rotate(45deg);
      }

      .comment-container {
         border: 1px solid #bbb;
         flex: 1;
         min-height: 100px;
         border-radius: 3px;
         overflow: hidden;
         background-color: #fff;

        .comment-title {
          width: 100%;
          height: 40px;
          background-color: #eee;
          border-bottom:  1px solid #bbb;
          line-height: 40px;

          .comment-nickname {
            font-weight: bold;
            color: #586069;
            font-size: 16px;
          }

          span {
            font-size: 14px;
            margin-left: 10px;
            color: #666;
          }
        }

        .comment-content {
          width: 100%;
          min-height: 60px;
          
          p {
            font-size: 14px;
            text-indent: 2em;
            margin: 5px;
          }
        }
      }
      
    }
    
  }

  .slide-fade-enter-active {
    transition: all .3s ease;
  }
  .slide-fade-leave-active {
    transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .slide-fade-enter, .slide-fade-leave-to
  /* .slide-fade-leave-active for below version 2.1.8 */ {
    transform: translateX(10px);
    opacity: 0;
  }

</style>
