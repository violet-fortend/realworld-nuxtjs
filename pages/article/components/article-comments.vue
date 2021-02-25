<template>
  <div>
    <form v-if="user" class="card comment-form" @submit.prevent="onSubmit">
      <div class="card-block">
        <textarea
          v-model="myComment.body"
          class="form-control"
          placeholder="Write a comment..."
          rows="3"
        ></textarea>
      </div>
      <div class="card-footer">
        <img v-if="user.image" :src="user.image" class="comment-author-img" />
        <button class="btn btn-sm btn-primary">Post Comment</button>
      </div>
    </form>
    <p v-else style="display: inherit;">
      <nuxt-link to="/login">Sign in</nuxt-link>or
      <nuxt-link to="/register">Sign up</nuxt-link>to add comments on this article.
    </p>
    <div class="card" v-for="comment in comments" :key="comment.id">
      <div class="card-block">
        <p class="card-text">{{comment.body}}</p>
      </div>
      <div class="card-footer">
        <nuxt-link
          :to="{
            name: 'profile',
            params: {
                username: comment.author.username
            }
        }"
          class="comment-author"
        >
          <img :src="comment.author.image" class="comment-author-img" />
        </nuxt-link>&nbsp;
        <nuxt-link
          :to="{
            name: 'profile',
            params: {
                username: comment.author.username
            }
        }"
          class="comment-author"
        >{{comment.author.username}}</nuxt-link>
        <span class="date-posted">{{comment.createdAt | date('MMM DD, YYYY')}}</span>
        <span
          class="mod-options"
          v-if="comment.author.username===user.username"
          @click="deleteComment(comment.id)"
        >
          <i class="ion-trash-a"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import { getComments, addComment, deleteComment } from '@/api/article'
import { mapState } from 'vuex'

export default {
  name: 'ArticleComments',
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      comments: [],
      myComment: {
        body: ''
      }
    }
  },
  computed: {
    ...mapState(['user'])
  },
  async mounted() {
    this.getComments()
  },

  methods: {
    async getComments() {
      const { data } = await getComments(this.article.slug)
      this.comments = data.comments
    },
    async onSubmit() {
      try {
        const res = await addComment(this.article.slug, {
          comment: this.myComment
        })
        //清空
        this.myComment.body = ''
        this.getComments()
      } catch (err) {}
    },
    async deleteComment(id) {
      await deleteComment(this.article.slug, id)
      this.getComments()
    }
  },

  watch: {}
}
</script>
<style>
</style>