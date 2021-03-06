<template lang="pug">
  wrapper(
    :category = 'category'
    :page = 'page'
    :course = 'conference'
    :account = 'account'
    :completedUnlogged = 'completedUnlogged'
    :current = 'current'
    :selected = 'selected'
    :lesson = 'talk',
    :restricted = 'restricted',
    :isLesson = 'false')
</template>


<script>
import { mapState } from 'vuex'
import wrapper from '~/components/lessons/Wrapper'
import meta from '~/mixins/meta'

export default {
  name: 'page-talk',

  middleware: 'anonymous',

  components: {
    wrapper
  },

  transition (from, to) {
    if (from && to) {
      const fromArray = from.path.split('/')
      const toArray = to.path.split('/')
      if (fromArray.length > 3 && toArray.length > 3) {
        return fromArray[2] === toArray[2] ? 'settings' : 'page'
      }
    }
    return 'page'
  },

  head () {
    const imageUrl = this.current.image ? this.current.image[0].url : ''
    return meta.get({
      categoryTitle: this.conference.title,
      categorySlug: this.category,
      pageSlug: this.current.slug,
      pageTitle: this.current.title,
      description: this.current.description,
      category: 'conferences',
      image: imageUrl,
      facebookImage: (this.current.facebookImage && this.current.facebookImage[0].url) || imageUrl,
      twitterImage: (this.current.twitterImage && this.current.twitterImage[0].url) || imageUrl,
      author: 'Adam Jahr'
    })
  },

  data () {
    return {
      category: this.$route.params.conference,
      page: this.$route.params.talk,
      selected: -1,
      restricted: true,
      current: {}
    }
  },

  watch: {
    account () {
      this.getContent()
    }
  },

  created () {
    if (this.conference) this.getContent()
  },

  computed: {
    ...mapState({
      conference: result => {
        result.courses.conference.lessons = result.courses.conference.talks
        return result.courses.conference
      },
      talk: result => result.courses.talk,
      account: result => result.account.account,
      completedUnlogged: result => result.account.completedUnlogged
    })
  },

  methods: {
    getContent () {
      // If no talk selected, get the first one of the conference
      if (this.page === null) this.page = this.conference.talks[0].slug

      this.conference.talks.map((talk, index) => {
        // Find the selected talk in the list
        if (this.page === talk.slug) {
          // Load the current talk
          this.current = talk
          // Keep track of talk index for the carousel
          this.selected = index
        }
      })

      this.loadContent()
    },

    loadContent () {
      this.checkRestriction()
      this.$store.dispatch('courses/getContent', {
        category: 'talks',
        slug: this.page,
        restricted: this.restricted
      })
    },

    checkRestriction () {
      // Talk don't have the free option
      let restriction = this.current.free !== undefined ? !this.current.free : false
      // Check lessons restrictions
      if (restriction) {
        restriction = this.account ? !this.account.subscribed : true
      } else if (this.current.lock) {
        restriction = !this.account
      }
      this.restricted = restriction
    }
  },

  async fetch ({ store, params }) {
    await store.dispatch('courses/getConference', params.conference)
  }
}
</script>
