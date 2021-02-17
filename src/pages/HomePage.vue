<template>
  <q-page class="constrain q-pa-md">
    <div class="row q-col-gutter-lg">
      <div class="col-12 col-sm-8">
        <!--   loopping threw every post     -->
        <template v-if="!loadingPosts && posts.length">
          <q-card v-for="post in posts"
                  :key="post.id"
                  class="card-post q-mb-md" flat bordered >
            <q-item>

              <q-item-section avatar class="space">
                <q-avatar>
                  <img src="../assets/eytanavatar.jpg">
                </q-avatar>
              </q-item-section>

              <q-item-section>
                <q-item-label class="text-bold">Eytan-Sayada</q-item-label>
                <q-item-label caption>
                  {{ post.location }}
                </q-item-label>
              </q-item-section>
              <q-item-section thumbnail>
                <q-btn @click="cardSettings" flat round color="grey-10" icon="eva-options-outline" />

              </q-item-section>

            </q-item>

            <q-separator/>
            <q-img
              :src="post.imgUrl"
            />
            <q-card-section>
              <div>{{ post.caption }}</div>
              <div class="text-caption text-grey"> {{ post.date | niceData }}</div>
            </q-card-section>
          </q-card>
        </template>

        <!-- if no posts at all..-->
        <template v-else-if="!loadingPosts && !posts.length">
          <h5 class="text-center text-grey">Sorry. No posts to desplay yet.</h5>
        </template>

        <!--   loading card     -->
        <template v-else>
          <q-card flat bordered>
            <q-item>
              <q-item-section avatar>
                <q-skeleton type="QAvatar" animation="fade" size="40px"/>
              </q-item-section>

              <q-item-section>
                <q-item-label>
                  <q-skeleton type="text" animation="fade"/>
                </q-item-label>
                <q-item-label caption>
                  <q-skeleton type="text" animation="fade"/>
                </q-item-label>
              </q-item-section>
            </q-item>

            <q-skeleton height="200px" square animation="fade"/>

            <q-card-actions align="right" class="q-gutter-md">
              <q-skeleton type="text" class="text-subtitle2" animation="fade"/>
              <q-skeleton type="text" width="50%" class="text-subtitle2" animation="fade"/>
            </q-card-actions>
          </q-card>
        </template>
        <!--  end of loading card  -->

      </div>
      <div class="col-4 large-screen-only">
        <q-item class="fixed">
          <q-item-section avatar>
            <q-avatar size="50px">
              <img src="../assets/eytanavatar.jpg">
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-bold">Eytan-Sayada</q-item-label>
            <q-item-label caption>
              Eytan Sayada
            </q-item-label>
          </q-item-section>
        </q-item>
      </div>
    </div>
  </q-page>
</template>

<script>
import {date} from 'quasar'

export default {
  name: 'HomePage',
  data() {
    return {
      posts: [],
      loadingPosts: false,
    }
  },
  methods: {
    getPosts() {
      this.loadingPosts = true
      this.$axios.get(`${process.env.API}/posts`).then(res => {
        this.posts = res.data
        this.loadingPosts = false
      }).catch(err => {
        this.$q.dialog({
          title: 'Alert',
          message: 'Something Went Wrong. Please Try Again'
        })
        this.loadingPosts = false
      })

    },
    cardSettings() {
      this.$q.dialog({
        title: 'Alert',
        message: 'ההגדרות בבנייה!'
      })
    },
  },
  filters: {
    // niceData its quasar plugin
    niceData(value) {
      return date.formatDate(value, 'MMMM D h:mmA')
    }
  },
  created() {
    this.getPosts()
  }
}
</script>


<style lang="scss">
.card-post {
  .q-img {
    min-height: 200px;
  }
}

</style>
