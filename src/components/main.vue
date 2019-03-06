<template>
  <div class="daily">
    <div class="daily-menu">
      <div class="daily-menu-item"
           @click="handleToRecommend"
           :class="{ on:type === 'recommend' }">每日推荐</div>
      <div class="daily-menu-item"
           :class="{ on:type === 'daily' }"
           @click="showThemes = !showThemes">主题日报</div>
      <ul v-show="showThemes">
        <li v-for="item in themes" :key="item.index">
          <a :class="{ on: item.id === themeId && type === 'daily'}"
             @click="handleToTheme(item.id)">{{item.name}}</a>
        </li>
      </ul>
    </div>
    <div class="daily-list" ref="list">
      <template v-if="type === 'recommend'">
        <div v-for="list in recommendList" :key="list.index">
          <div class="daily-date">{{formatDay(list.date)}}</div>
          <Item v-for="item in list.stories" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
        </div>
      </template>
      <template v-if="type === 'daily'">
        <Item v-for="item in list" :data="item" :key="item.id" @click.native="handleClick(item.id)"></Item>
      </template>
    </div>
    <daily-article :id="articleId"></daily-article>
  </div>
</template>

<script>
import $ from '../../libs/utils'
import Item from "./item"
import dailyArticle from './daily-article'
export default {
  name: 'HelloWorld',
  components: {Item, dailyArticle},
  data () {
    return {
      type: 'recommend',
      recommendList: [],
      dailyTime: $.getTodayTime(),
      isLoading: false,
      themes: [],
      list: [],
      showThemes: false,
      themeId: 0,
      articleId: 0
    }
  },
  methods: {
    handleToRecommend () {
      this.type = 'recommend'
      this.recommendList = []
      this.dailyTime = $.getTodayTime()
      this.getRecommendList()
    },
    getRecommendList () {
      this.isLoading = true
      const prevDay = $.prevDay(this.dailyTime + 86400000)
      $.ajax.get('news/before/' + prevDay).then(res => {
        this.recommendList.push(res)
        this.isLoading = false
      })
    },
    // 转换为带汉子的月日
    formatDay (date) {
      let month = date.substr(4, 2)
      let day = date.substr(6, 2)
      if (month.substr(0, 1) === '0') month = month.substr(1, 1)
      if (day.substr(0, 1) === '0') day = day.substr(1, 1)
      return `${month} 月 ${day} 日`
    },
    handleToTheme (id) {
      // 改变菜单分类
      this.type = 'daily'
      // 设置当前点击子类的主题日报id
      this.themeId = id
      // 清空中间栏的数据
      this.list = []
      $.ajax.get('theme/' + id).then(res => {
        // 过滤掉类型为1的文章，该类型下的文章为空
        this.list = res.stories.filter(item => item.type !== 1)
      })
    },
    getThemes () {
      // axios 发起get请求
      $.ajax.get('themes').then(res => {
        this.themes = res.others
      })
    },
    handleClick (id) {
      this.articleId = id
    }
  },
  mounted () {
    // 初始化时调用
    this.getThemes()
    this.getRecommendList()
    // 获取到DOM
    const $list = this.$refs.list
    // 监听中栏的滚动事件
    $list.addEventListener('scroll', () => {
      // 在主题日报或正在加载推荐列表时停止操作
      if (this.type === 'daily' || this.isLoading) return
      // 已经滚动的距离加页面的高度等于整个内容区域高度时，视为接触底部
      if
      (
        $list.scrollTop + document.body.clientHeight >= $list.scrollHeight
      ) {
        // 时间相对减少一天
        this.dailyTime -= 86400000
        this.getRecommendList()
      }
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  html, body{
    margin: 0;
    padding: 0;
    height: 100%;
    color: #657180;
    font-size: 16px;
  }
  .daily-menu{
    width: 150px;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    overflow: auto;
    background: #f5f7f9;
  }
  .daily-menu-item{
    font-size: 18px;
    text-align: center;
    margin: 5px 0;
    padding: 10px 0;
    cursor: pointer;
    border-right: 2px solid transparent;
    transition: all .3s ease-in-out;
  }
  .daily-menu-item:hover{
    background: #e3e8ee;
  }
  .daily-menu-item.on{
    border-right: 2px solid #3399ff;
  }
  .daily-list{
    width: 300px;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 150px;
    overflow: auto;
    border-right: 1px solid #d7dde4;
  }
  .daily-date{
    text-align: center;
    margin: 10px 0;
  }
  .daily-item{
    display: block;
    color: inherit;
    text-decoration: none;
    padding: 16px;
    overflow: hidden;
    cursor: pointer;
    transition: all .3s ease-in-out;
  }
  .daily-item:hover{
    background: #e3e8ee;
  }
  .daily-article{
    margin-left: 450px;
    padding: 20px;
  }
  .daily-menu ul{
    list-style: none;
  }
  .daily-menu ul li a{
    display: block;
    color: inherit;
    text-decoration: none;
    padding: 5px 0;
    margin: 5px 0;
    cursor: pointer;
  }
  .daily-menu ul li a:hover, .daily-menu ul li a.on{
    color: #3399ff;
  }
</style>
