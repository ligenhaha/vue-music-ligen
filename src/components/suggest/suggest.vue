<template>
  <scroll class="suggest"
          :data="result"
          :pullup="pullup"
          @scrollToEnd="searchMore"
          ref="suggest">
    <ul class="suggest-list">
      <li class="suggest-item" v-for="(item,index) in result" :key="index">
        <div class="icon">
          <i :class="getIconCls(item)"></i>
        </div>
        <div class="name">
          <p class="text" v-html="getDisplayName(item)"></p>
        </div>
      </li>
      <loading v-show="hasMore" title=""></loading>
    </ul>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import {search} from 'api/search'
  import {ERR_OK} from 'api/config'
  import Scroll from 'base/scroll/scroll'
  // import {filterSinger} from 'common/js/song'
  import {createSong, isValidMusic, processSongsUrl} from 'common/js/song'
  import Loading from 'base/loading/loading'

  const perpage = 20
  const TYPE_SINGER = 'singer'
  export default {
    props: {
      query: {
        type: String,
        default: ''
      },
      showSinger: {
        type: Boolean,
        default: true
      }
    },
    data () {
      return {
        page: 1,
        result: [],
        pullup: true,
        hasMore: true
      }
    },
    methods: {
      search () {
        this.hasMore = true
        this.page = 1
        this.$refs.suggest.scrollTo(0, 0)
        search(this.query, this.page, this.showSinger, perpage).then((res) => {
          if (res.code === ERR_OK) {
            this._genResult(res.data).then((result) => {
              this.result = result
            })
            this._checkMore(res.data)
          }
        })
      },
      searchMore () {
        if (!this.hasMore) {
          return
        }
        this.page++
        search(this.query, this.page, this.showSinger, perpage).then((res) => {
          if (res.code === ERR_OK) {
            this._genResult(res.data).then((result) => {
              this.result = this.result.concat(result)
            })
            this._checkMore(res.data)
          }
        })
      },
      getIconCls (item) {
        if (item.type === TYPE_SINGER) {
          return 'icon-mine'
        } else {
          return 'icon-music'
        }
      },
      getDisplayName (item) {
        if (item.type === TYPE_SINGER) {
          return item.singername
        } else {
          return `${item.name}-${item.singer}`
        }
      },
      _checkMore (data) {
        const song = data.song
        if (!song.list.length || (song.curnum + song.curpage * 20) >= song.totalnum) {
          this.hasMore = false
        }
      },
      _genResult (data) {
        let ret = []
        if (data.zhida && data.zhida.singerid && this.page === 1) {
          ret.push({...data.zhida, ...{type: TYPE_SINGER}})
        }
        return processSongsUrl(this._normalizeSongs(data.song.list)).then((songs) => {
          ret = ret.concat(songs)
          return ret
        })
      },
      _normalizeSongs (list) {
        let ret = []
        list.forEach((musicData) => {
          if (isValidMusic(musicData)) {
            ret.push(createSong(musicData))
          }
        })
        return ret
      }
    },
    watch: {
      query () {
        this.search()
      }
    },
    components: {
      Scroll,
      Loading
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .suggest
    height: 100%
    overflow: hidden

    .suggest-list
      padding: 0 30px

      .suggest-item
        display: flex
        align-items: center
        padding-bottom: 20px

      .icon
        flex: 0 0 30px
        width: 30px

        [class^="icon-"]
          font-size: 14px
          color: $color-text-d

      .name
        flex: 1
        font-size: $font-size-medium
        color: $color-text-d
        overflow: hidden

        .text
          no-wrap()

    .no-result-wrapper
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
