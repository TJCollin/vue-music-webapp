<template>
  <div class="song-list">
    <ul>
      <li v-for="(song, index) in songs" :key="song.id" class="item" @click="selectItem(song, index)">
        <!-- <div class="rank" v-show="rank">
          <span :class="getRankCls(index)">{{getRankText(index)}}</span>
        </div> -->
        <p class="count">{{index + 1}}</p>
        <div class="content">
          <h2 class="name">{{song.name}}</h2>
          <p class="desc">{{getDesc(song)}}</p>
        </div>
      </li>
    </ul>
  </div>
</template>


<script>
	export default {
		name: "SongList",  props: {
			songs: {
				type: Array
			}
		},
		methods: {
			selectItem (item, index) {
				this.$emit('select', item, index)
			},
			getDesc (song) {
				if (song.aliaName) {
					return `${song.singer} - ${song.aliaName}`
				} else {
					return `${song.singer}`
				}
			}
		}
	}
</script>

<style lang="scss" scoped>
  @import "../../assets/scss/variable";
  @import "../../assets/scss/mixin";
  .song-list {
    .item {
      position: relative;
      display: flex;
      align-items: center;
      box-sizing: border-box;
      height: px2rem(60px);
      border-bottom: 1px solid rgb(228, 228, 228);
      .count {
        flex: 0 0 px2rem(50px);
        font-size: px2rem(16px);
        width: px2rem(50px);
        text-align: center;
        color: $color-text-g;
      }
      .content {
        flex: 1;
        line-height: px2rem(20px);
        overflow: hidden;
        font-size: px2rem(16px);
        .name {
          margin-top: px2rem(4px);
          width: 80%;
          @include no-wrap();
          color: $color-text;
        }
        .desc {
          @include no-wrap();
          // margin-top: 3px;
          width: 80%;
          font-size: px2rem(12px);
          color: $color-text-g;
        }
      }
    }
  }
</style>