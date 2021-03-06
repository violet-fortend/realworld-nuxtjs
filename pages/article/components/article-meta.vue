<template>
  <div class="article-meta">
    <nuxt-link
      :to="{
        name: 'profile',
        params: {
            username: article.author.username
        }
    }"
    >
      <img :src="article.author.image" />
    </nuxt-link>
    <div class="info">
      <nuxt-link
        class="author"
        :to="{
        name: 'profile',
        params: {
            username: article.author.username
        }
    }"
      >{{article.author.username}}</nuxt-link>
      <span class="date">{{article.createAt | date('MMM DD, YYYY')}}</span>
    </div>

    <template v-if="user && user.username === article.author.username">
      <nuxt-link class="btn btn-outline-secondary btn-sm" :to="`/editor/${article.slug}`">
        <i class="ion-edit"></i> Edit Article
      </nuxt-link>

      <button class="btn btn-outline-danger btn-sm" @click="deleteArticle">
        <i class="ion-trash-a"></i> Delete Article
      </button>
    </template>

    <template v-else>
      <button
        class="btn btn-sm btn-outline-secondary"
        :class="{active: article.author.following}"
        :disabled="disabled_follow"
        @click="handleFollow"
      >
        <i class="ion-plus-round"></i>
        &nbsp;
        {{article.author.following ? 'Unfollow' : 'Follow'}} {{article.author.username}}
      </button>
      &nbsp;&nbsp;
      <button
        class="btn btn-sm btn-outline-primary"
        :class="{active: article.favorited}"
        :disabled="disabled_favorite"
        @click="handleFavorite"
      >
        <i class="ion-heart"></i>
        &nbsp;
        {{article.favorited ? 'Unfavorite' : 'Favorite'}} Article
        <span
          class="counter"
        >({{article.favoritesCount}})</span>
      </button>
    </template>
  </div>
</template>

<script>
import { addFollow, deleteFollow } from '@/api/profile'
import { addFavorite, deleteFavorite, deleteArticle } from '@/api/article'
import { mapState } from 'vuex'
export default {
  name: 'ArticleMeta',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  computed: {
    ...mapState(['user'])
  },
  data() {
    return {
      disabled_follow: false,
      disabled_favorite: false
    }
  },
  methods: {
    async deleteArticle() {
      await deleteArticle(this.article.slug)
      this.$router.replace('/')
    },
    async handleFollow() {
      try {
        this.disabled_follow = true
        const { following, username } = this.article.author
        if (following) {
          await deleteFollow(username)
          this.article.author.following = false
        } else {
          await addFollow(username)
          this.article.author.following = true
        }
        this.disabled_follow = false
      } catch (err) {
        this.disabled_follow = false
      }
    },
    async handleFavorite() {
      try {
        this.disabled_favorite = true
        const { favorited, slug } = this.article
        if (favorited) {
          await deleteFavorite(slug)
          this.article.favoritesCount += -1
          this.article.favorited = false
        } else {
          await addFavorite(slug)
          this.article.favoritesCount += 1
          this.article.favorited = true
        }
        this.disabled_favorite = false
      } catch (err) {
        this.disabled_favorite = false
      }
    }
  }
}
</script>

<style>
</style>