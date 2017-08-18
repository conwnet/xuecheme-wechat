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

          <mt-cell class="part" title="已选套餐" is-link @click.native="packSheetVisible = !packSheetVisible">{{ pack.name }}</mt-cell>

          <div class="part panel">
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
              <mt-field label="姓名" placeholder="怎么称呼您？" v-model="name"></mt-field>
            </div>
          </mt-popup>

          <mt-actionsheet :actions="packNames" v-model="packSheetVisible">
          </mt-actionsheet>
      </mt-tab-container-item>

    </mt-tab-container>
  </div>
</template>

<script>

// import { Toast } from 'mint-ui'

export default {
  name: 'app',
  data () {
    return {
      page: '1', // NavBar 的选项值
      packSheetVisible: false,
      enrollPopupVisible: false,
      pack: {},
      logo: {},
      packs: [],
      banners: []
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
  methods: {
  },
  created () {
    setTimeout(() => {
      this.banners = [
        { src: 'http://placehold.it/350x150', link: '#' },
        { src: 'http://placehold.it/350x150', link: '#' }
      ]
      this.logo = { src: 'http://placehold.it/80x80' }
      this.packs = [
        { name: '2588 平民套餐', desc: '2588 平民套餐，一费到底，快速拿证！', price: 2588, id: 1 },
        { name: '2888 奢华套餐', desc: '2888 奢华套餐，科二保过，拿证更容易！', price: 2588, id: 2 },
        { name: '3388 土豪套餐', desc: '3388 土豪套餐，科二科三直飞，极速拿证！', price: 2588, id: 3 }
      ]
      this.pack = this.packs[1]
    }, 500)
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
    height: 150px;
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
  }


</style>
