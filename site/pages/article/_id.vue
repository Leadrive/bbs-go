<template>
  <section class="main">
    <div class="container main-container size-360 left-main">
      <div class="left-container">
        <article
          class="article-item article-detail"
          itemscope
          itemtype="http://schema.org/BlogPosting"
        >
          <div class="main-content">
            <div class="article-header">
              <div class="article-item-left">
                <a
                  :href="'/user/' + article.user.id"
                  :title="article.user.nickname"
                  target="_blank"
                >
                  <div
                    :style="{
                      backgroundImage: 'url(' + article.user.avatar + ')'
                    }"
                    class="avatar is-rounded"
                  />
                </a>
              </div>
              <div class="article-item-right">
                <h1 class="article-title" itemprop="headline">
                  {{ article.title }}
                </h1>
                <div class="article-meta">
                  <span class="article-meta-item">
                    由
                    <a
                      :href="'/user/' + article.user.id"
                      class="article-author"
                      itemprop="author"
                      itemscope
                      itemtype="http://schema.org/Person"
                      ><span itemprop="name">{{
                        article.user.nickname
                      }}</span></a
                    >发布于
                    <time
                      :datetime="
                        article.createTime | formatDate('yyyy-MM-ddTHH:mm:ss')
                      "
                      itemprop="datePublished"
                      >{{ article.createTime | prettyDate }}</time
                    >
                  </span>

                  <span
                    v-if="article.tags && article.tags.length > 0"
                    class="article-meta-item"
                  >
                    <span
                      v-for="tag in article.tags"
                      :key="tag.tagId"
                      class="article-tag tag"
                    >
                      <a :href="'/articles/' + tag.tagId" class>{{
                        tag.tagName
                      }}</a>
                    </span>
                  </span>
                </div>

                <div class="article-tool">
                  <span v-if="isOwner">
                    <a @click="deleteArticle(article.articleId)">
                      <i class="iconfont icon-delete" />删除
                    </a>
                  </span>
                  <span v-if="isOwner">
                    <a :href="'/article/edit/' + article.articleId">
                      <i class="iconfont icon-edit" />修改
                    </a>
                  </span>
                  <span>
                    <a @click="addFavorite(article.articleId)">
                      <i class="iconfont icon-favorite" />{{
                        favorited ? '已收藏' : '收藏'
                      }}
                    </a>
                  </span>
                </div>
              </div>
            </div>

            <div class="ad">
              <!-- 展示广告 -->
              <adsbygoogle ad-slot="1742173616" />
            </div>

            <div
              v-html="article.content"
              class="article-content content"
              itemprop="articleBody"
            ></div>
          </div>

          <div class="article-footer">
            <ul>
              <li>
                <strong>免责声明：</strong>
                我们尊重原创，也注重分享。版权原作者所有，如有侵犯您的权益请及时联系（
                <a href="mailto:mlog1@qq.com">mlog1@qq.com</a
                >），我们将在24小时之内删除。
              </li>
              <li v-if="article.sourceUrl">
                <strong>原文链接：</strong>
                <a
                  :href="article.sourceUrl"
                  class="source-url"
                  rel="nofollow"
                  target="_blank"
                  >{{ article.sourceUrl }}</a
                >
              </li>
            </ul>
          </div>

          <div class="ad">
            <!-- 展示广告 -->
            <adsbygoogle ad-slot="1742173616" />
          </div>

          <!-- 评论 -->
          <comment
            :entity-id="article.articleId"
            :comments-page="commentsPage"
            :show-ad="true"
            entity-type="article"
          />
        </article>
      </div>
      <div class="right-container">
        <div style="max-height: 360px;">
          <!-- 展示广告 -->
          <adsbygoogle ad-slot="1742173616" />
        </div>

        <div
          v-if="relatedArticles && relatedArticles.length"
          class="widget no-margin"
        >
          <div class="widget-header">相关文章</div>
          <div class="widget-content article-related">
            <ul>
              <li v-for="a in relatedArticles" :key="a.articleId">
                <a
                  :href="'/article/' + a.articleId"
                  :title="a.title"
                  class="article-related-title"
                  target="_blank"
                  >{{ a.title }}</a
                >
              </li>
            </ul>
          </div>
        </div>

        <div style="max-height: 360px;">
          <!-- 展示广告 -->
          <adsbygoogle ad-slot="1742173616" />
        </div>

        <div
          v-if="newestArticles && newestArticles.length"
          class="widget no-margin"
        >
          <div class="widget-header">最新文章</div>
          <div class="widget-content article-related">
            <ul>
              <li v-for="a in newestArticles" :key="a.articleId">
                <a
                  :href="'/article/' + a.articleId"
                  :title="a.title"
                  class="article-related-title"
                  target="_blank"
                  >{{ a.title }}</a
                >
              </li>
            </ul>
          </div>
        </div>

        <div class="ad" style="max-height: 360px;">
          <!-- 展示广告 -->
          <adsbygoogle ad-slot="1742173616" />
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import utils from '~/common/utils'
import Comment from '~/components/Comment'

export default {
  components: {
    Comment
  },
  async asyncData({ $axios, params, error }) {
    let article
    try {
      article = await $axios.get('/api/article/' + params.id)
    } catch (e) {
      error({
        statusCode: 404,
        message: '文章不存在，或已被删除'
      })
      return
    }
    const [
      commentsPage,
      favorited,
      newestArticles,
      relatedArticles
    ] = await Promise.all([
      $axios.get('/api/comment/list', {
        params: {
          entityType: 'article',
          entityId: article.articleId
        }
      }),
      $axios.get('/api/favorite/favorited', {
        params: {
          entityType: 'article',
          entityId: params.id
        }
      }),
      $axios.get('/api/article/user/newest/' + article.user.id),
      $axios.get('/api/article/related/' + article.articleId)
    ])

    // 文章关键字
    let keywords = ''
    const keywordsArr = []
    if (article.tags && article.tags.length > 0) {
      article.tags.forEach((tag) => {
        keywordsArr.push(tag.tagName)
      })
      if (keywordsArr.length > 0) {
        keywords = keywordsArr.join(',')
      }
    }

    // 文章描述
    let description = ''
    if (article.summary && article.summary.length > 0) {
      description = article.summary.substr(0, 128)
      if (article.summary.length > 128) {
        description += '...'
      }
    }

    return {
      article,
      favorited: favorited.favorited,
      newestArticles,
      relatedArticles,
      commentsPage,
      keywords,
      description
    }
  },
  computed: {
    isOwner() {
      return (
        this.$store.state.user.current &&
        this.article &&
        this.$store.state.user.current.id === this.article.user.id
      )
    }
  },
  methods: {
    async deleteArticle(articleId) {
      if (process.client && !window.confirm('是否确认删除该文章？')) {
        return
      }
      try {
        await this.$axios.post('/api/article/delete/' + articleId)
        this.$toast.success('删除成功！', {
          duration: 1000,
          onComplete() {
            utils.linkTo('/articles')
          }
        })
      } catch (e) {
        this.$toast.error('删除失败：' + (e.message || e))
      }
    },
    async addFavorite(articleId) {
      try {
        if (this.favorited) {
          await this.$axios.get('/api/favorite/delete', {
            params: {
              entityType: 'article',
              entityId: articleId
            }
          })
          this.favorited = false
          this.$toast.success('已取消收藏！')
        } else {
          await this.$axios.post('/api/article/favorite/' + articleId)
          this.favorited = true
          this.$toast.success('收藏成功！')
        }
      } catch (e) {
        console.error(e)
        this.$toast.error('收藏失败：' + (e.message || e))
      }
    }
  },
  head() {
    return {
      title: this.$siteTitle(this.article.title),
      meta: [
        { hid: 'description', name: 'description', content: this.description },
        { hid: 'keywords', name: 'keywords', content: this.keywords }
      ]
    }
  }
}
</script>

<style lang="scss" scoped></style>
