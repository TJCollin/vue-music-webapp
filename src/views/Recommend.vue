<template>
  <div class="recommend" ref="recommend">
    <scroll class="recommend-content" ref="scroll" :data="playList">
      <div>
        <div v-show="banner.length" class="decorate"></div>

        <div v-if="banner.length" class="slider-wrapper">

          <slider>
            <div v-for="item in banner" :key="item.encodeId" @click.stop="selectBanner(item)">
              <img :src="item.imageUrl">
            </div>
          </slider>
        </div>
        <div class="recommend-list" ref="recommendList">
          <h1 class="title">推荐歌单</h1>
          <ul>
            <li class="item" v-for="item in playList" :key="item.id">
              <div class="icon" @click="selectList(item)">
                <div class="gradients"></div>
                <img v-lazy="item.picUrl">
              </div>
              <p class="play-count">
                <i class="fa fa-headphones"></i>
                {{Math.floor(item.playCount / 10000) }}万
              </p>
              <div class="text">
                <p class="name">{{item.name}}</p>
              </div>
            </li>
          </ul>
        </div>
        <div class="recommend-song" ref="recommendSong">
          <h1 class="title">推荐歌曲</h1>
          <ul>
            <li class="item" v-for="item in recommendMusic" :key="item.id" @click="selectSong(item)">
              <div class="icon">
                <img v-lazy="item.image">
              </div>
              <p class="text">{{item.name}}</p>
              <p class="singer">{{item.singer}}</p>
            </li>
          </ul>
        </div>
      </div>
    </scroll>
    <router-view></router-view>
  </div>
</template>


<script>
	import Scroll from "../components/base/Scroll";
	import Slider from "../components/base/Slider";
	import {getBanner, getRecommendMusic, getRecommendList} from "../common/api/recommend";
	import {getSongDetail} from "../common/api/search";
	import {createRecommendSong} from "../common/js/Song";
	import {ERR_OK} from "../common/js/config";
	import {playlistMixin} from "../common/js/mixin";
	import {mapMutations, mapActions} from 'vuex'

	export default {
		name: "Recommend",
		mixins: [playlistMixin],
		data() {
			return {
				banner: [],
				playList: [],
				recommendMusic: []
			}
		},
		created() {
			this._getBanner()
			this._getRecommendList()
			this._getRecommendMusic()
			window.addEventListener('resize', () => {
				let bannerBack = [...this.banner]
        this.banner =  [];
				setTimeout(() => {
					this.banner = bannerBack
        }, 20)
				// this.banner = []
				// console.log('resize')
				// this._getBanner()
			})
			// this.$refs.recommendList.style.
		},
		methods: {
			// firstPlay () {
			//   console.log('firstPlay')
			//   this.$refs.audio.play()
			// },
			selectBanner(item) {
				let regHttp = /^http/
				let regSong = /\/song\?id/
				if (regHttp.test(item.url)) {
					window.open(item.url)
				}
				if (regSong.test(item.url) || item.targetId) {
					getSongDetail(item.targetId).then((res) => {
						let m = res.data.songs[0]
						let song = {
							id: m.id,
							singer: m.ar[0].name,
							name: m.name,
							image: m.al.picUrl,
							album: m.al.name
						}
						this.insertSong(song)
						this.setFullScreen(true)
					})
				}
			},
			selectSong(item) {
				this.insertSong(item)
			},
			handlePlaylist(playlist) {
				const bottom = playlist.length > 0 ? '60px' : ''
				this.$refs.recommend.style.bottom = bottom
				this.$refs.scroll.refresh()
			},
			selectList(item) {
				this.$router.push({
					path: `/recommend/${item.id}`
				})
				// console.log(item)
				this.setMuiscList(item)
			},
			_getBanner() {
				getBanner().then((res) => {
					if (res.status === ERR_OK) {
						let list = res.data.banners
						this.banner = list.splice(4)
					} else {
						console.error('Banner 获取失败')
					}
				})
			},
			_getRecommendList() {
				getRecommendList().then((res) => {
					if (res.status === ERR_OK) {
						this.playList = res.data.result
					} else {
						console.error('getRecommendList 获取失败')
					}
				})
			},
			_getRecommendMusic() {
				getRecommendMusic().then((res) => {
					if (res.status === ERR_OK) {
						let list = res.data.result.map((item) => {
							return createRecommendSong(item)
						})
						list.splice(9)
						this.recommendMusic = list
					} else {
						console.error('getRecommendMusic 获取失败')
					}
				})
			},
			...mapMutations({
				setMuiscList: 'SET_MUSIC_LIST',
				setFullScreen: 'SET_FULL_SCREEN'
			}),
			...mapActions([
				'insertSong'
			])
		},
		components: {
			Slider,
			Scroll
		}
	}
</script>

<style lang="scss" scoped>
  @import "../assets/scss/variable";
  @import "../assets/scss/mixin";
  @import "../assets/scss/function";
  .recommend {
    position: fixed;
    width: 100%;
    top: px2rem(88px);
    bottom: 0;
    z-index: 100;
    .recommend-content {
      width: 100%;
      height: 100%;
      overflow: hidden;
      /*overflow 无效 注意position relative*/
      position: relative;
      .decorate {
        position: absolute;
        top: -32vh;
        z-index: -10;
        background: $color-theme;
        width: 100%;
        height: 50vh;
        vertical-align: inherit;
      }
      .slider-wrapper {

        width: 96%;
        margin: 0 auto;
        border-radius: 5px;
        overflow: hidden;
        position: relative;
      }
      .recommend-list {
        position: relative;
        box-sizing: border-box;
        width: 100%;
        text-align: center;
        .title {
          height: 65px;
          line-height: 65px;
          padding-left: 1.5%;
          text-align: left;
          font-size: $font-size-medium;
          font-weight: bold;
          color: $color-text;
        }
        .item {
          display: inline-block;
          position: relative;
          box-sizing: border-box;
          width: 33%;
          padding: 0 1%;
          .icon {
            position: relative;
            display: inline-block;
            width: 100%;
            margin-bottom: px2rem(5px);
            .gradients {
              position: absolute;
              top: 0;
              width: 100%;
              height: px2rem(35px);
              border-radius: px2rem(3px);
              background: linear-gradient(rgba(109, 109, 109, 0.4),rgba(255, 255, 255, 0));
            }
            img {
              width: 100%;
              height: 100%;
              border-radius: px2rem(3px);
            }
          }
          .play-count {
            position: absolute;
            top: px2rem(5px);
            right: px2rem(8px);
            font-size: $font-size-small-s;
            color: $color-text-l
          }
          .text {
            float: left;
            line-height: px2rem(16px);
            text-align: left;
            height: px2rem(40px);
            line-height: px2rem(16px);
            overflow: hidden;
            margin-bottom: px2rem(10px);
            font-size: $font-size-small;
          }
        }
      }
      .recommend-song {
        margin-top: px2rem(-20px);
        box-sizing: border-box;
        width: 100%;
        text-align: center;
        .title {
          height: px2rem(65px);
          line-height: px2rem(65px);
          padding-left: 1.5%;
          text-align: left;
          font-size: $font-size-medium;
          font-weight: bold;
          color: $color-text;
        }
        .item {
          display: inline-block;
          position: relative;
          box-sizing: border-box;
          width: 33%;
          padding: 0 1%;
          .icon {
            position: relative;
            display: inline-block;
            width: 100%;
            margin-bottom: px2rem(5px);
            img {
              width: 100%;
              height: 100%;
              border-radius: px2rem(3px);
            }
          }
          .text {
            line-height: px2rem(16px);
            text-align: left;
            height: px2rem(16px);
            @include no-wrap();
            font-size: $font-size-small;
          }
          .singer {
            line-height: px2rem(16px);
            margin-bottom: px2rem(10px);
            text-align: left;
            @include no-wrap();
            font-size: $font-size-small;
            color: $color-text-g;
          }
        }
      }
    }
  }

</style>