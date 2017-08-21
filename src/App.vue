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
                <h1>学车么- 携手银河驾校</h1>
                <p class="price">
                  <span>¥</span>
                  <span style="font-size: 30px;">{{ pack.price || '???' }}</span>
                </p>
              </div>
            </div>
            
            
            <div style="display: flex; color: #888; font-size: 12px; margin-top: 10px;">
              <div style="flex: 1; text-align:center;">已报名：2</div>
              <div style="flex: 1; text-align:center;">评价：21</div>
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
              <mt-field label="微信" placeholder="获取失败，请重新进入" v-model="nickname" readonly disableClear></mt-field>
              <mt-field label="姓名" placeholder="怎么称呼您？" :state="checkFormat('name') ? 'success' : 'error'" v-model="name"></mt-field>
              <mt-field label="手机号" placeholder="怎么联系您？" :state="checkFormat('mobile') ? 'success' : 'error'" v-model="mobile"></mt-field>
              <mt-field label="验证码" placeholder="请查收短信～" :state="checkFormat('verify') ? 'success' : 'error'" v-model="verify">
                <mt-button type="primary" @click="sendVerify()" :disabled="verifyButton.disabled">{{ verifyButton.text }}</mt-button>
              </mt-field>
              <mt-field label="推荐人" placeholder="推荐人的手机号" v-model="invite"></mt-field>
            </div>
            <div class="part">
              <mt-button style="width: 95%; margin: 0 auto;" type="primary" size="large" @click="enrollPopupVisible = true">微信支付</mt-button>
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
            <mt-field type="textarea" rows="3" style="border: 1px solid #bbb;"></mt-field>
            <mt-button size="small" type="primary" class="fb-btn">发表</mt-button>
            <mt-button size="small" class="fb-btn" @click="inputBox = !inputBox">取消</mt-button>
          </div>
        </transition>



        <mt-spinner type="snake" color="#26a2ff" v-if="!comments.length" :size="40" style="display: block; width: 40px; margin: 0 auto; padding: 10px;"></mt-spinner>

        <transition-group name="slide-fade">
          <div class="comment" v-for="(item, index) in comments" :key="'comment-' + index" v-if="showComment(item.state)">
            <img class="comment-headimg" :src="item.headimgurl" />
            <div class="comment-arrow"></div>
            <div class="comment-container">
              <div class="comment-title">
                <span class="comment-nickname">{{ item.nickname }}</span>
                <span>{{ item.date }}</span>
                <span>({{ item.state ? '已报名' : '未报名' }})</span>
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

// import { Toast } from 'mint-ui'

export default {
  name: 'app',
  data () {
    return {
      page: '', // NavBar 的选项值
      messagePopupVisible: false,
      // enroll page
      packSheetVisible: false,
      enrollPopupVisible: false,
      message: '',
      nickname: '',
      name: '',
      mobile: '',
      verify: '',
      invite: '',
      pack: {},
      logo: {},
      packs: [],
      banners: [],
      verifyButton: { disabled: false, text: '立即发送' },
      // comment page
      comments: [],
      inputBox: false,
      alreadyEnroll: true,
      notEnroll: false
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
        setTimeout(() => {
          this.banners = [
            { src: 'http://placehold.it/350x200', link: '#' },
            { src: 'http://placehold.it/350x200', link: '#' }
          ]
          this.logo = { src: 'http://placehold.it/80x80' }
          this.packs = [
            { name: '2588 平民套餐', desc: '2588 平民套餐，一费到底，快速拿证！', price: 2588, id: 1 },
            { name: '2888 奢华套餐', desc: '2888 奢华套餐，科二保过，拿证更容易！', price: 2888, id: 2 },
            { name: '3388 土豪套餐', desc: '3388 土豪套餐，科二科三直飞，极速拿证！', price: 3388, id: 3 }
          ]
          this.nickname = '凌乱'
          this.pack = this.packs[1]
        }, 1000)
      } else if (val === 'comment') {
        setTimeout(() => {
          this.comments = [
            {
              headimgurl: 'http://placehold.it/80x80',
              nickname: '凌乱',
              date: '2017-08-15',
              state: 0,
              content: '后来使用开发工具查看是用CSS来实现的,现记录'
            },
            {
              headimgurl: 'http://placehold.it/80x80',
              nickname: '别急还没硬',
              date: '2017-08-15',
              state: 1,
              content: '预处理技术现在已经非常成熟,比较流行的有Less'
            },
            {
              headimgurl: 'http://placehold.it/80x80',
              nickname: '凌乱',
              date: '2017-08-15',
              state: 0,
              content: '后来使用开发工具查看是用CSS来实现的,现记录'
            },
            {
              headimgurl: 'http://placehold.it/80x80',
              nickname: '别急还没硬',
              date: '2017-08-15',
              state: 1,
              content: '预处理技术现在已经非常成熟,比较流行的有Less'
            }
          ]
        }, 1000)
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
      this.Message('发送成功！')
      let second = 60
      this.verifyButton = { disabled: true, text: '60s' }
      let intval = setInterval(() => {
        this.verifyButton = { disabled: true, text: --second + 's' }
      }, 1000)
      setTimeout(() => {
        clearInterval(intval)
        this.verifyButton = { disabled: false, text: '再次发送' }
      }, 60000)
    },
    showComment (state) {
      return (state && this.alreadyEnroll) || (!state && this.notEnroll)
    },
    checkFormat (type) {
      if (type === 'name') {
        return this.name !== ''
      } else if (type === 'mobile') {
        return /1[0-9]{10}/.test(this.mobile) && this.mobile.length === 11
      } else if (type === 'verify') {
        return /[0-9]+/.test(this.verify)
      }
    }
  },
  created () {
    this.page = 'enroll'
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
