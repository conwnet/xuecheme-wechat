<template>
  <div>
    <mt-navbar v-model="page">
      <mt-tab-item id="1">套餐详情</mt-tab-item>
      <mt-tab-item id="2">学员评价</mt-tab-item>
      <mt-tab-item id="3">在线客服</mt-tab-item>
    </mt-navbar>

    <mt-tab-container id="container" v-model="page">
      <mt-tab-container-item id="1">
         <mt-swipe id="banner" :auto="4000">
            <mt-swipe-item v-for="(item, index) in banners" :key="index"><img :src="item.src" /></mt-swipe-item>
          </mt-swipe>
          <div class="panel">
            <div style="display: flex">
              <div style="width: 80px; height: 80px;"><img :src="logo.src" /></div>
              <div style="flex: 1; margin-left: 10px;">学车么- 携手银河驾校</div>
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
              <mt-field label="姓名" placeholder="怎么称呼您？" v-model="user.name"></mt-field>
              <mt-field label="手机号" placeholder="怎么联系您？" v-model="user.mobile"></mt-field>
              <mt-field label="验证码" placeholder="请查收短信～" v-model="user.name">
                <mt-button type="primary" @click="sendVerify()" :disabled="verifyButton.disabled">{{ verifyButton.text }}</mt-button>
              </mt-field>
              <mt-field label="推荐人" placeholder="推荐人的手机号" v-model="user.mobile"></mt-field>
            </div>
            <div class="part">
              <mt-button style="width: 95%; margin: 0 auto;" type="primary" size="large" @click="enrollPopupVisible = true">微信支付</mt-button>
            </div>
          </mt-popup>

          <mt-actionsheet :actions="packNames" v-model="packSheetVisible">
          </mt-actionsheet>
      </mt-tab-container-item>

      <mt-tab-container-item id="2">
        <div class="comments">
            <mt-spinner type="snake" color="#26a2ff" v-if="!comments.length" :size="40" style="display: block; width: 40px; margin: 0 auto; padding: 10px;"></mt-spinner>
            <div class="comment" v-for="(item, index) in comments" :key="index">
              <div class="headimg"><img :src="item.headimgurl"></div>
            </div>
        </div>
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
      message: '',
      nickname: '',
      enrollPopupVisible: false,
      user: {},
      pack: {},
      logo: {},
      packs: [],
      banners: [],
      verifyButton: { disabled: false, text: '立即发送' },
      // comment page
      comments: []
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
      if (val === '1') {
        setTimeout(() => {
          this.banners = [
            { src: 'http://placehold.it/350x200', link: '#' },
            { src: 'http://placehold.it/350x200', link: '#' }
          ]
          this.logo = { src: 'http://placehold.it/80x80' }
          this.packs = [
            { name: '2588 平民套餐', desc: '2588 平民套餐，一费到底，快速拿证！', price: 2588, id: 1 },
            { name: '2888 奢华套餐', desc: '2888 奢华套餐，科二保过，拿证更容易！', price: 2588, id: 2 },
            { name: '3388 土豪套餐', desc: '3388 土豪套餐，科二科三直飞，极速拿证！', price: 2588, id: 3 }
          ]
          this.nickname = '凌乱'
          this.pack = this.packs[1]
        }, 2000)
      } else if (val === '2') {
        setTimeout(() => {
          this.comments = [
            {
              headimgurl: 'http://placehold.it/80x80',
              nickname: '凌乱'
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
    }
  },
  created () {
    this.page = '2'
  }
}
</script>

<style>
  body {
    margin: 0;
    background-color: #eee;
  }

  #container {
    margin-top: 3px;
  }

  #banner {
    width: 100%;
    height: 200px;
  }

  .panel {
    padding: 5px;
    background-color: #fff;
  }

  .confimPage {
    width: 100%;
    height: 100%;
  }

  #banner img {
    width: 100%;
    height: 100%;
  }

  .part {
    margin-top: 10px;
    background-color: #fff;
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

  .comments {
    background-color: #fff;
  }

  .comment .headimg {
    width: 50px;
    height: 50px;
  }

  .comment .headimg img {
    width: 100%;
    height: 100%;
  }
</style>
