<template>
  <div class="search-box">
    <input
      @input="query = $event.target.value"
      :aria-label="inputLabel"
      :placeholder="inputLabel"
      :value="query"
      :class="{ 'focused': focused }"
      autocomplete="off"
      spellcheck="false"
      @focus="focused = true"
      @blur="focused = false"
      @keyup.enter="go(focusIndex)"
      @keyup.up="onUp"
      @keyup.down="onDown"
    >
    <ul
      class="suggestions"
      v-if="showSuggestions"
      :class="{ 'align-right': alignRight }"
      @mouseleave="unfocus"
    >
      <li
        class="suggestion"
        v-for="(s, i) in suggestions"
        :class="{ focused: i === focusIndex }"
        @mousedown="go(i)"
        @mouseenter="focus(i)"
      >
        <a :href="s.path" @click.prevent>
          <span class="page-title">{{ s.title || s.path }}</span>
          <span v-if="s.header" class="header">&gt; {{ s.header.title }}</span>
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: "",
      focused: false,
      focusIndex: 0
    };
  },
  created() {
    this.query = this.inputLabel;
  },

  watch: {
    focused(focused) {
      if (focused) {
        this.query = "";
      } else {
        if (!this.query) {
          this.query = this.inputLabel;
        }
      }
    },
    inputLabel() {
      this.query = this.inputLabel;
    }
  },
  computed: {
    showSuggestions () {
      return (
        this.focused &&
        this.suggestions &&
        this.suggestions.length
      )
    },

    suggestions () {
      const query = this.query.trim().toLowerCase();
      if (!query && query !== this.inputLabel) {
        return
      }

      const { pages, themeConfig } = this.$site
      const max = themeConfig.searchMaxSuggestions || 5;
      const localePath = this.$localePath
      const matches = item => (
        item.title &&
        item.title.toLowerCase().indexOf(query) > -1
      )
      const res = []
      for (let i = 0; i < pages.length; i++) {
        if (res.length >= max) break
        const p = pages[i]
        // filter out results that do not match current locale
        if (this.getPageLocalePath(p) !== localePath) {
          continue
        }
        if (matches(p)) {
          res.push(p)
        } else if (p.headers) {
          for (let j = 0; j < p.headers.length; j++) {
            if (res.length >= max) break
            const h = p.headers[j]
            if (matches(h)) {
              res.push(Object.assign({}, p, {
                path: p.path + '#' + h.slug,
                header: h
              }))
            }
          }
        }
      }
      return res
    },

    // make suggestions align right when there are not enough items
    alignRight () {
      const navCount = (this.$site.themeConfig.nav || []).length
      const repo = this.$site.repo ? 1 : 0
      return navCount + repo <= 2
    },

    inputLabel () {
      return this.$themeLocaleConfig.search || 'Search';
    }
  },

  methods: {
    getPageLocalePath (page) {
      for (const localePath in this.$site.locales || {}) {
        if (localePath !== '/' && page.path.indexOf(localePath) === 0) {
          return localePath
        }
      }
      return '/'
    },

    onUp () {
      if (this.showSuggestions) {
        if (this.focusIndex > 0) {
          this.focusIndex--
        } else {
          this.focusIndex = this.suggestions.length - 1
        }
      }
    },

    onDown () {
      if (this.showSuggestions) {
        if (this.focusIndex < this.suggestions.length - 1) {
          this.focusIndex++
        } else {
          this.focusIndex = 0
        }
      }
    },

    go (i) {
      if (!this.showSuggestions) {
        return
      }
      this.$router.push(this.suggestions[i].path)
      this.query = ''
      this.focusIndex = 0
    },

    focus (i) {
      this.focusIndex = i
    },

    unfocus () {
      this.focusIndex = -1
    }
  }
}
</script>

<style lang="stylus">
@import '../styles/config.styl'

.search-box
  display inline-block
  position relative
  margin-right 1rem
  height 100%
  input
    cursor text
    width 12.8rem
    box-sizing border-box
    color #fff
    display inline-block
    border none
    border-radius 2rem
    font-size 0.9rem
    line-height 2.5rem
    padding 0 2.3rem 0 1.1rem
    outline none
    transition all .2s ease
    background rgba(0,0,0, 0.15) url(../search.svg) 93% center no-repeat
    background-size 1rem
    height 100%
    &:focus
      cursor auto
      border-color #fff
    &::placeholder
      color rgba(255, 255, 255, 0.5)
  .suggestions
    background #fff
    width 20rem
    position absolute
    top 1.5rem
    border 1px solid darken($borderColor, 10%)
    border-radius 6px
    padding 0.4rem
    list-style-type none
    &.align-right
      right 0
  .suggestion
    line-height 1.4
    padding 0.4rem 0.6rem
    border-radius 4px
    cursor pointer
    a
      white-space normal
      color lighten($textColor, 35%)
      .page-title
        font-weight 600
      .header
        font-size 0.9em
        margin-left 0.25em
    &.focused
      background-color #f3f4f5
      a
        color $mainColor

@media (max-width: $MQNarrow)
  .search-box
    input
      cursor pointer
      width 0
      border-color transparent
      position relative
      padding 0 22px
      background-position center center
      background-color transparent
      &:focus
        background-color rgba(0,0,0, 0.15)
        background-position  93% center
        padding 0 2.3rem 0 1.1rem
        cursor text
        left 0
        width 100%

@media (max-width: $MQNarrow) and (min-width: $MQMobile)
  .search-box
    .suggestions
      left 0

@media (max-width: $MQMobile)
  .search-box
    margin-right 0
    input
      left 0
    .suggestions
      right 0

@media (max-width: $MQMobileNarrow)
  .search-box
    .suggestions
      width calc(100vw - 4rem)
    input:focus
      width 100%
</style>
