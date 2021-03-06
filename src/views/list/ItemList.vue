<template lang='pug'>
  .view
    .toolbox
      .sorter
        a.sort-item(@click='sortItems("views")') Top
        a.sort-item(@click='sortItems("time")') New
      form.search-box(@click='searching = true' @submit.prevent='findItems')
        input.search(type='search' placeholder='Droids you\'re looking for...'
          v-model.lazy='searchedTags'  :class='{ "search--active": searching }'
          @blur='clearSearch' @keyup.esc='clearSearch'
          pattern='[a-zA-Z0-9- ]+' maxlength='50' required
          title='Droid\'s serial number can contain only alphanumeric characters, dashes and spaces.')
    transition-group.gallery(name='translate-up' tag='div')
      item(v-for='(item, index) in displayedItems'  :key='item.id'
        @click.native='fetchItemView(item.id, index)'  :item='item')
    mugen-scroll(:handler='loadItems'
      :should-handle='!loading && hasMore') {{ loadingText }}
    transition(name='slide-up' appear)
      btn-top

    transition-group(name='fade')
      .item-view-overlay(@click.self='activeId = ""' v-if='activeId'  ref='overlay' key='overlay')
        item-view(:id='activeId' @scroll='scrollOverlay' ref='view')
      a.item-view-esc.material-icons(@click='activeId = ""' v-if='activeId' key='close') close
      a.item-view-arrow.material-icons.arrow-prev(@click='changeActiveId(-1)'
        v-if='activeId' key='left') keyboard_arrow_left
      a.item-view-arrow.material-icons.arrow-next(@click='changeActiveId(1)'
        v-if='activeId' key='right') keyboard_arrow_right
</template>

<script>
import MugenScroll from 'vue-mugen-scroll'
import Item from '@/components/Item.vue'
import ItemView from '../single/ItemView.vue'
import BtnTop from '@/components/BtnTop.vue'

export default {
  name: 'item-list',

  components: {
    Item,
    ItemView,
    BtnTop,
    MugenScroll
  },

  props: {
    type: String
  },

  data() {
    return {
      displayedItems: this.$store.getters.activeItems,
      activeId: '',
      activeIndex: -1,
      searchedTags: '',
      searching: false,
      loading: false
    }
  },

  computed: {
    slice() {
      return this.$store.state.activeSlice
    },
    maxSlice() {
      const { itemsPerSlice, lists } = this.$store.state
      return Math.ceil(lists[this.type].length / itemsPerSlice)
    },
    hasMore() {
      return this.slice < this.maxSlice
    },
    loadingText() {
      return this.hasMore ? 'keep searching...' : 'the end of the road'
    }
  },

  beforeMount() {
    if (this.$root._isMounted) {
      this.sortItems('views')
    }
  },

  watch: {
    activeId() {
      this.activeId
      ? document.documentElement.classList.add('hide-scrollbar')
      : document.documentElement.classList.remove('hide-scrollbar')
    }
  },

  methods: {
    loadItems() {
      this.$bar.start()
      this.loading = true
      this.$store.commit('INCREMENT_ACTIVE_SLICE')
      this.$store.dispatch('ENSURE_ACTIVE_ITEMS').then(() => {
        this.displayedItems = this.$store.getters.activeItems
      })
      this.loading = false
      this.$bar.finish()
    },
    sortItems(sort) {
      this.$bar.start()
      this.$store.commit('SET_ACTIVE_SORT', { sort })
      this.$store.dispatch('FETCH_LIST_DATA', {
        type: this.type
      }).then(() => {
        this.displayedItems = this.$store.getters.activeItems
      })
      this.$bar.finish()
    },
    clearSearch() {
      this.searching = false
      this.searchedTags = ''
    },
    findItems() {
      const tags = this.searchedTags.split(' ')
      this.$bar.start()
      this.$store.dispatch('FETCH_SEARCHED_LIST', {
        type: this.type,
        tags
      }).then(() => {
        this.displayedItems = this.$store.getters.activeItems
      })
      this.$bar.finish()
    },
    fetchItemView(id, index) {
      this.$store.dispatch('FETCH_ITEM_VIEW', { id }).then(() => {
        if (this.type === 'articles') {
          this.$router.push(`/articles/${id}`)
        } else {
          document.documentElement.classList.add('hide-scrollbar')
          this.activeIndex = index
          this.activeId = id
        }
      })
    },
    closeView() {
      document.documentElement.classList.remove('hide-scrollbar')
      this.activeId = ''
    },
    changeActiveId(direction) {
      const newIndex = this.activeIndex + direction
      this.activeId = this.displayedItems[newIndex].id
      this.activeIndex = newIndex
    },
    scrollOverlay() {
      const commentEl = this.$refs.view.$refs.comments.$el
      TweenLite.to(this.$refs.overlay, 0.5, { scrollTo: commentEl })
    }
  }
}
</script>

<style lang='scss'>
@import '~style';

.view {
  background-color: $color-white;
}
.toolbox {
  position: relative;
  background-color: $color-main;
  padding: 3rem 5rem;
  box-shadow: 0px 4px 30px rgba(0, 0, 0, 0.25);
  display: flex;
  justify-content: flex-end;
  align-items: center;
  @include screen-style(iphone7) {
    padding: 3rem 2rem;
  };
  @include screen-style(iphoneSE) {
    padding: 3rem 2rem;
  };
}
.sorter {
  display: flex;
  @include screen-style(iphone7) {
    position: absolute;
    width: 100%;
    justify-content: center;
    left: 0;
    bottom: -5rem;
    background-color: $color-lightblue;
  };
  @include screen-style(iphoneSE) {
    position: absolute;
    width: 100%;
    justify-content: center;
    left: 0;
    bottom: -5rem;
    background-color: $color-lightblue;
  };
}
.sort-item {
  @extend %btn-text;
  width: 10rem;
  height: 5rem;
  margin-right: 2rem;
  color: $color-white;
  line-height: 5rem;
  background-color: $color-blue;
  @include screen-style(iphone7) {
    background-color: transparent;
    box-shadow: none;
    border-radius: 0;
    margin-right: 0;
    color: $color-blue;
  };
  &:hover {
    background-color: $color-white;
  }
  @include screen-style(iphoneSE) {
    background-color: transparent;
    box-shadow: none;
    border-radius: 0;
    margin-right: 0;
    color: $color-blue;
  };
  &:hover {
    background-color: darken($color-blue, 10%);
  }
}
.sort-item--on {
  opacity: 0.2;
  transition: 0.3s ease-in-out;
}
.search-box {
  position: relative;
  cursor: pointer;
  &::before {
    content: '';
    position: absolute;
    top: calc(50% - 2.5rem / 2);
    left: 1.3rem;
    width: 2.5rem;
    height: 2.5rem;
    background: url('~@/assets/icons/search.svg') no-repeat center / 100%;
  }
}
.search {
  appearance: none;
  background-color: $color-white;
  color: $color-main;
  padding: 0 2rem;
  width: 5rem;
  height: 5rem;
  border-radius: 40px;
  border-style: none;
  outline-style: none;
  font-size: 1.6rem;
  transition: all .5s;
  cursor: pointer;
  @include screen-style(fullHd) {
    width: 30rem;
    padding: 0 5rem;
  };
}
.search::placeholder {
  opacity: 0;
}
.search--active {
  width: 30rem;
  padding: 0 5rem;
  cursor: text;
  @include screen-style(iphone7) {
    width: 27rem;
  };
  @include screen-style(iphoneSE) {
    width: 22rem;
  };
}
.search--active::placeholder {
  opacity: 1;
}
.gallery {
  display: flex;
  justify-content: flex-start;
  flex-wrap: wrap;
  padding: 6rem 5rem 0rem 5rem;
  @include screen-style(iphone7) {
    padding: 8rem 2rem;
  };
  @include screen-style(iphoneSE) {
    padding: 8rem 2rem;
  };
}
.mugen-scroll {
  display: block;
  text-align: center;
  text-transform: capitalize;
  height: 5rem;
  font-size: 1.5rem;
  font-weight: 600;
  color: darken($color-grey, 20%);
}
.item-view-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
  overflow-y: scroll;
  background-color: rgba(51, 51, 51, 0.8);
}
.item-view-esc {
  position: absolute;
  width: 5rem;
  height: 5rem;
  z-index: 1001;
  font-size: 4rem;
  text-align: center;
  line-height: 5rem;
  color: $color-white;
  right: 1rem;
  top: 1rem;
  opacity: 0.8;
  transition: all 0.3s ease-in-out;
  cursor: pointer;
  background: transparent;
  &:hover {
    opacity: 1;
    transition: all 0.3s ease-in-out;
  }
  @media (orientation: portrait) {
    right: 2rem;
    top: 3rem;
    background-color: $color-blue;
    opacity: 1;
    border-radius: 50%;
  }
}
.item-view-arrow {
  position: fixed;
  top: 30rem;
  width: 7rem;
  height: 7rem;
  z-index: 1001;
  font-size: 5rem;
  line-height: 7rem;
  color: $color-white;
  opacity: 0.8;
  text-align: center;
  cursor: pointer;
  outline: none;
  transition: 0.3s ease-in-out;
  &:hover {
    opacity: 1;
    transition: 0.3s ease-in-out;
  }
  @include screen-style(ipadPro) {
    display: none;
  };
  @media (orientation: portrait) {
    display: none;
  }
}
.arrow-prev {
  left: 16vw;
}
.arrow-next {
  right: 16vw;
}

.translate-up-move, .translate-up-enter-active, .translate-up-leave-active {
  transition: all .5s cubic-bezier(.55,0,.1,1);
}
.translate-up-enter {
  opacity: 0;
  transform: translateY(30px);
}
.translate-up-leave-active {
  opacity: 0;
  transform: translateY(-30px);
}
</style>
