<template lang="html">
  <div class="article-detail-wrap">
      <div class="by-article">
          <div class="by-article-header">
              <h1 class="by-article-title">{{ m_article_title }}</h1>
              <div class="by-author-info">
                <img class="by-avatar" v-bind:src="m_author_picture || '/static/img/default_avatar.jpg'" alt="作者头像"/>
                <p class="name-and-time">
                  <a href="#" class="article-author author-name">{{ m_author_name }}</a>·<span class="article-time">{{ m_check_date | timeFormat }}</span>·<span class="comment-num">{{m_comment_num}}评论</span>
                  <span class="like-num">{{m_like_num}}喜欢</span>
                  <span class="share-num">{{m_share_num}}分享</span>
                </p>
                <!-- <router-link :to="{ path: '/category/'+ m_category.toLowerCase() }" :class="'article-type '+ m_category.toLowerCase()">{{ m_category_full_name }}</router-link> -->
              </div>
          </div>
          <div class="by-article-content simditor-article" v-html="m_article_content"></div>
      </div>
      <div class="by-article-footer">
        <span class="article-copyright">© 版权归作者所有，转载请联系冰岩作坊</span>
        <span class="like-wrap select-no" data-tip="点赞" :class="{'active': m_liked}" v-on:click="f_like">
          <b class="iconfont icon-like1"></b>|
          <b class="like-num">{{ m_like_num }}</b>
        </span>
        <div class="page-turn">
          <router-link :to="{path: '/article/' + m_pre_article_id}" class="prev" :class="{'disable-page': !m_pre_article_id }">上一篇</router-link>
          <router-link :to="{path: '/article/' + m_next_article_id}" class="next" :class="{'disable-page': !m_next_article_id }">下一篇</router-link>
        </div>
      </div>
      <Comment :article-id='m_article_id' :refresh="f_get_comments"></Comment>
      <comment-render :comments="m_comments" :refresh="f_get_comments"></comment-render>
  </div>
</template>

<script>
import Comment from './Comment.vue'
import CommentRender from './CommentRender.vue'
export default {
  data () {
    return {
      m_comments: [], // 获取文章的评论内容
      m_article_id: null,
      m_article_title: '',
      m_article_date: '',
      m_check_date: '',
      m_author_name: '',
      m_author_picture: null,
      m_share_num: 0,
      m_comment_num: 0,
      m_like_num: 0,
      m_article_cover: '',
      m_article_content: '',
      m_category_full_name: '',
      m_category: '',
      m_pre_article_id: null,
      m_next_article_id: null,
      m_liked: false,
      m_share_weixin_show: false,
      m_qrcode_src: null
    }
  },
  mounted () {
    this.f_get_article()
    this.f_get_comments()
  },
  watch: {
    '$route': ['f_get_article', 'f_get_comments']
  },
  methods: {
    f_set_share: function (cover, word) {
      let title = document.querySelector('title')
      let img = document.querySelector('#share-img')
      img.setAttribute('src', 'http://ued.bingyan.net' + cover)
      title.innerHTML = word
    },
    f_get_article: function () {
      let id = this.$route.params.id
      this.$http.get('/api/article/' + id).then(function (response) {
        let body
        try {
          body = JSON.parse(response.body)
        } catch (err) {
          // console.warn(说明返回的已经是以json的格式的了)
          body = response.body
        }
        let article = body.data.article
        this.m_article_id = article.article_id       // id
        this.m_check_date = article.check_date   // 审核通过的日期
        this.m_article_title = article.title   // 标题
        this.m_article_cover = article.cover   // 封面图片
        this.m_article_content = article.content   // 正文
        this.m_author_name = article.author_name   // 作者名称
        this.m_author_picture = article.author_picture   // 作者头像
        this.m_category_full_name = article.category_full_name   // 类别全称
        this.m_category = article.category   // 类别全称
        this.m_comment_num = article.comment_num   // 评论数量
        this.m_like_num = article.like_num   // 喜欢的数量
        this.m_share_num = article.share_num   // 分享数量

        this.m_pre_article_id = body.data.article_pre
        this.m_next_article_id = body.data.article_next
        this.m_liked = body.data.liked

        this.f_set_share(this.m_article_cover, this.m_article_title)
      })
    },
    f_get_comments: function () {
      let id = this.$route.params.id
      this.$http.get('/api/article/' + id + '/comments').then(function (response) {
        this.m_comments = response.body.data
        // 处理评论区内容的表情  过滤函数不能用只能这样处理了。。
        this.m_comments.forEach(function (item) {
          item.comment_content = this.$formatEmoji(item.comment_content)
        }.bind(this))
      })
    },
    f_like: function () {
      this.$http.post('/api/like', {article_id: this.m_article_id})
        .then(function (response) {
          let body = response.body
          if (body.status === 1) {
            this.m_liked = true
            this.m_like_num = body.data
          } else if (body.status === 2) {
            this.$warn(body.msg)
          } else if (body.status === 3) {
            this.$warn('您已经点过赞了')
          }
        })
    },
    f_share_weixin: function () {
      if (this.m_share_weixin_show) {
        return
      }
      if (this.m_qrcode_src) {
        return (this.m_share_weixin_show = true)
      }
      this.$http.post('/api/qrcode', {
        article_id: this.m_article_id
      }).then(function (response) {
        if (response.body.status === 2) {
          return this.$warn('获取文章二维码失败')
        }
        this.m_share_weixin_show = true
        this.m_qrcode_src = response.body.data
      })
    },
    f_not_share_weixin: function () {
      this.m_share_weixin_show = false
    }
  },
  components: {
    Comment, CommentRender
  }
}
</script>

<style lang="scss" scoped>
@import '../../scss/_varilables.scss';
@import '../../scss/_mixin.scss';
.article-detail-wrap{
  background-color: #fff;
  position: relative;
  padding:20px 15px 10px;
}
.by-article-content{
  padding-top:20px;
  padding-bottom: 40px;
}
.by-article-footer{
  border-radius:2px;
  position: relative;
  height:40px;
  line-height: 40px;
  .article-copyright{
    font-size: 12px;
    color: #999;
    position: absolute;
    top:-36px;
    right:0;
  }
  .like-wrap{
    cursor: pointer;
    display: inline-block;
    width: 80px;
    height:30px;
    line-height: 30px;
    text-align: center;
    border: 1px solid $bingyan-color;
    color: $bingyan-color;
    border-radius: 15px;
    &:hover{
      background: lighten($bingyan-color,15%) !important;
      border-color: lighten($bingyan-color,15%) !important;
      color:#fff;
    }
    &.active{
      background: $bingyan-color;
      border-color: $bingyan-color;
      color:#fff;
    }
    &:active{
      background: $bingyan-color !important;
      border-color: $bingyan-color !important;
      color:#fff;
    }
    b{
      height:30px;
      display: inline-block;
      width:30px;
    }
  }
  .page-turn{
    height:40px;
    line-height: 40px;
    position: absolute;
    right:0;
    top:0;
    a{
      padding-top: 10px;
      padding-bottom: 10px;
      color: #666;
      &:hover{
        color: #000;
        text-decoration: underline;
      }
      padding-left: 15px;
      &.disable-page{
        font-size:0;
        padding:0;
      }
    }
  }
}
//文章头部
.by-article-header{
  position: relative;
  border-bottom: 1px solid lighten($light-black,35%);
  .by-article-title{
    font-size: 24px;
    line-height: 1.2;
    font-weight: bold;
  }
  .by-author-info{
    color:$baseTextColor;
    height: 50px;
    line-height: 50px;
    .by-avatar{
      width:40px;
      height:40px;
      border-radius: 50%;
      vertical-align: middle;
    }
    .name-and-time{
      display: inline-block;
      height:50px;
      .author-name{
        color:$bingyan-color;
      }
      .article-time,.comment-num,.like-num,.share-num{
        font-size: 12px;
      }
    }
    .article-type{
      float: right;
      color:#fff;
      padding:0 8px;
      margin-top: 13px;
      line-height: 2;
      font-size: 12px;
      text-align: center;
      border-radius: 2px;
      @each $item,$color in $nav-list{
        &.#{$item}{
          background-color: $color;
        }
      }
    }
  }
}
</style>
