<template>
  <transition name="fade">
    <div class="find">
      <!-- 搜索框 -->
      <div class="search">
        <input type="search" placeholder="歌曲、歌手、专辑" v-model="msg" @focus="Focus" @keyup.enter="search(msg)">
        <i v-show="isShowkey" @click="Blur">取消</i>
      </div>
      <!-- 热门搜索 -->
      <div class="hotSearch" v-show="!isShowkey">
        <div class="hot">热门搜索</div>
        <div class="hotList">
          <p v-for="item in hotkey" @click="search(item.k)">{{ item.k }}</p>
        </div>
      </div>
      <!-- 搜索历史 -->
      <div class="searchHistory" v-show="!isShowkey &&  searchHistory[0]">
        <div class="hot">搜索历史<span class="iconfont icon-del" @click="clearHistory"></span></div>
        <div class="historyList" ref="history">
          <ul>
          <p v-for="item in searchHistory" @click="search(item)">
            {{ item }}
            <span class="iconfont icon-del_1" @click.stop="delHistory(item)"></span>
          </p>
          </ul>
        </div>
      </div>
      <!-- 搜索结果 -->
        <div class="singer" @click="searchSingerAndAlbum(singer.mid,'singer')" v-show="isShowkey && singer.name">
          歌手：{{ singer.name }}
        </div>
        <div class="album" @click="searchSingerAndAlbum(album.mid,'album')" v-show="isShowkey && album.name">
          专辑：{{ album.name }}
        </div>
      <scroll :data="musics" @scroll="Bblur()" class="sResult" v-show="isShowkey">
        <song-list :songs="musics"></song-list>
      </scroll>
      <router-view></router-view>
    </div>
  </transition>
</template>

<script>
 import {mapGetters, mapMutations, mapActions} from 'vuex';
 import SongList from '../../base/song-list.vue';
 import Scroll from '../../base/scroll.vue';
 import {getHotKey,search} from '../../api/search.js';
 import {handleSong} from '../../base/song.js';
 import {clearAll} from '../../api/localStorage.js';
  export default {
    components: {
          SongList,
          Scroll
        },
    data() {
        return {
          msg: '',
          musics: [],
          hotkey:[],
          isShowkey: false,
          singer: {},
          album: {}
        };
    },
    computed: {
      ...mapGetters([
          'searchHistory'
        ])
    },
    created() {
      this._gethotKey();
    },
    methods: {
      ...mapMutations([
        'addHistory',
        'setDetailTypes',
        'setDetailMid',
        'clearHistory',
        'delHistory'
      ]),
      ...mapActions([
        'savePlayHistory'
      ]),
      // 当焦点在输入框， 滚动搜索结果时使输入框失去焦点
      // 因为在移动端浏览器中，焦点在输入框时，点击或滚动将呼出键盘
      Bblur() {
        document.activeElement.blur();
      },
      // 搜索
      search(msg) {
        msg = msg.trim();
        this.msg = msg;
            search(msg, 1, false, 15).then((res) => {
              if(res.code === 0) {
                this.musics = this.handleList(res.data.song.list);
                this.isShowkey = true;
                this.singer = res.data.song.list["0"].singer[0];
                this.album = {
                  mid: res.data.song.list["0"].albummid,
                  name: res.data.song.list["0"].albumname };
                this.addHistory(msg);
              }
      })
    },
    // 详情跳转
    searchSingerAndAlbum(id,type) {
      this.setDetailTypes(type);
      this.setDetailMid(id);
      this.$router.push({
            path:`/detail`
          });
    },
    // 遍历返回的数据，做数据提取处理
      handleList(list) {
        const List = [];
        list.forEach((item) => {
          let music = handleSong(item)
          List.push(music);
        });
        return List;
      },
      Focus() {
        this.isShowkey = true;
      },
      Blur() {
        this.isShowkey = false;
        this.musics = [];
        this.singer = {};
        this.album = {};
      },
      // 获取热搜
      _gethotKey() {
        getHotKey().then((res) => {
          if (res.code === 0) {
            this.hotkey = res.data.hotkey.splice(0,12);
          }
        })
      },
  }
}
</script>

<style scopeId>
@import './font_find/iconfont.css'
.find {
  position: absolute;
  width: 100%;
  top: 72px;
  bottom: 3.8em;
  overflow: hidden;
  background-color: #fff;
}
.search {
    width: 100%;
    padding: 5px 0;
    height: 46px;
    background-color: rgb(235, 237, 239);
    border-bottom: 1px solid rgba(1,186,144,0.8);
    display: flex;
}
.search input {
    flex: 1;
    border-radius: 5px;
    border: none;
    height: 30px;
    background-color: #fff;
    margin: 7px 30px 30px;
    text-indent: 25px;
}
.search i {
    height: 46px;
    line-height: 46px;
    flex: 0 0 60px;
    width: 60px;
}
.hotSearch {
  width: 100%;
  margin-top: 20px;
}
.hot {
  display: block;
  padding: 0px 0 10px 30px;
  position: relative;
}
.hotList {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  padding-left: 10px;
}
.hotList > p {
  border-radius: 10px;
  background-color:rgb(235, 237, 239);
  padding: 6px 10px;
  margin: 5px 12px;
  font-size: 13px;
}
.searchHistory {
  margin-top: 30px;
}
.historyList {
  height: 300px;
  overflow: hidden;
}
.historyList > ul > p {
  padding: 10px 30px;
  font-size: 14px;
}
.sResult {
  position: absolute;
  width: 100%;
  top: 198px;
  bottom: 1px;
  overflow: hidden;
}
.singer, .album {
  width: 100%;
  padding: 10px 0 10px 20px;
  font-size: 11px;
  border-bottom: 1px solid rgba(1,186,144,0.8);
}

.find .icon-del, .find .icon-del_1 {
  display: inline-block;
  width: 18px;
  height: 18px;
  font-size: 18px!important;
  float: right;
  color: black;
}

.find .icon-del {
  right: 15px;
}

</style>