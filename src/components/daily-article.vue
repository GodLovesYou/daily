<template>
    <div class="daily-article">
      <div class="daily-article-title">{{data.title}}</div>
      <div class="daily-article-content" v-html="data.body"></div>
      <div class="daily-comments" v-show="comments.length">
        <span>评论({{comments.length}})</span>
        <div class="daily-comment-avatar">
          <img :src="comment.avatar">
        </div>
        <div class="daily-comment-content">
          <div class="daily-comment-name">{{comment.author}}</div>
          <div class="daily-comment-time" v-time="comment.time"></div>
          <div class="daily-comment-text">{{comment.content}}</div>
        </div>
      </div>
    </div>
</template>

<script>
import Time from '../../directives/time'
import $ from '../../libs/utils'
export default {
  directives: {Time},
  props: {
    id: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      data: {},
      comments: [],
      comment: ''
    }
  },
  methods: {
    getArticle () {
      $.ajax.get('news/' + this.id).then(res => {
        // 将文章中的图片地址替换为代理地址
        res.body = res.body.replace(/src="http/g, 'src="' + $.imgPath + 'http')
        res.body = res.body.replace(/src="https/g, 'src="' + $.imgPath + 'https')
        this.data = res
        window.scrollTo(0, 0)
        this.getComments()
      })
    },
    getComments () {
      this.comments = []
      $.ajax.get('story/' + this.id + '/short-comments').then(res => {
        this.comments = res.comments.map(comment => {
          // 将头像的图片地址为代理地址
          comment.avatar = $.imgPath + comment.avatar
          return comment
        })
      })
    }
  },
  watch: {
    id (val) {
      if (val) this.getArticle()
    }
  }
}
</script>

<style scoped>
  .daily-article{
    margin-left: 450px;
    padding: 20px;
  }
  .daily-article-title{
    font-size: 28px;
    font-weight: bold;
    color: #222;
    padding: 10px 0;
  }
  .view-more a{
    display: block;
    cursor: pointer;
    background: #f5f7f9;
    text-align: center;
    color: inherit;
    text-decoration: none;
    padding: 4px 0;
    border-radius: 3px;
  }
</style>
