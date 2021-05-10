<template>
  <div>
 
    <div class="wrap">
    <!-- 播放器主体区域 -->
    <div class="play_wrap" id="player">
      <div class="search_bar">
        <img src="./assets/player_title.png" alt="" />
        <!-- 搜索歌曲 -->
        <input type="text" autocomplete="off" v-model="query" @keyup.enter="searchMusic" />
      </div>
      <div class="center_con">
        <!-- 搜索歌曲列表 -->
        <div class='song_wrapper'>
          <ul class="song_list">
            <li v-for="item in musicList">
              <a href="javascript:;" @click="playMusic(item.id)"></a> 
              <b>{{ item.name }}</b> 
              <span v-if="item.mvid!=0" @click="playMV(item.mvid)"><i></i></span>
            </li>
          </ul>
          <img src="./assets/line.png" class="switch_btn" alt="">
        </div>
        <!-- 歌曲信息容器 -->
        <div class="player_con" :class="{playing:isPlaying}">
          <img src="./assets/player_bar.png" class="play_bar" />
          <!-- 黑胶碟片 -->
          <img src="./assets/disc.png" class="disc autoRotate" />
          <img :src="musicCover" class="cover autoRotate" />
        </div>
        <!-- 评论容器 -->
        <div class="comment_wrapper">
          <h5 class='title'>热门留言</h5>
          <div class='comment_list'>
            <dl v-for="item in hotComments">
              <dt><img :src="item.user.avatarUrl" alt=""></dt>
              <dd class="name">{{ item.nickname}}</dd>
              <dd class="detail">
                {{ item.content }}
              </dd>
            </dl>
          </div>
          <img src="./assets/line.png" class="right_line">
        </div>
      </div>
      <div class="audio_con">
        <audio ref='audio' @play="play" @pause="pause" :src="musicUrl" controls autoplay loop class="myaudio"></audio>
      </div>
      <div class="video_con" v-show="isShow" style="display: none;">
        <video :src="mvUrl" controls="controls"></video>
        <div class="mask" @click="hide"></div>
      </div>
    </div>
  </div>
    
  
  </div>
</template>
<script>
import axios from 'axios'
import {reactive, toRefs} from '@vue/composition-api'
export default {
  name: 'App',
  //Vue3.0的核心方法
  // 回顾：setup方法
  // 1.  自动执行->在created方法调用之前执行
  // 2.  作用->为当前组件提供数据或者方法
  // 3.  合并到组件的data中
  // 4.  形参props context
  // 5.  return{数据的名字：值}

  setup(){
    //react方法提供的数据是响应式数据，相当于data
    const state = reactive({ 
      query: '',
      musicList:[],
      musicUrl: '',
      musicCover: '',
      hotComments: [],
      isPlaying: false,
      mvUrl: '',
      isShow: false
      })  //state->{query:''}

    //方法
    // 搜索音乐
    const searchMusic = async () => {
      const{ data } = await axios.get(
        'https://autumnfish.cn/search?keywords=' + state.query
        )
      state.musicList = data.result.songs
    }

    //播放音乐
    const playMusic = async musicId => {
      //音乐播放
      const{ data } = await axios.get(
        'https://autumnfish.cn/song/url?id=' + musicId
        )
      state.musicUrl = data.data[0].url

       // 封面展示
      const { data: datadetail } = await axios.get(
        'https://autumnfish.cn/song/detail?ids=' + musicId
      )
      state.musicCover = datadetail.songs[0].al.picUrl

      //热门评论
      const { data: dataCommet } = await axios.get(
        'https://autumnfish.cn/comment/hot?type=0&id=' + musicId
      )
      state.hotComments = dataCommet.hotComments;
      
    }
 
   const play = ()=> {
     state.isPlaying = true
   }

   const pause = ()=> {
     state.isPlaying = false
   }

   const hide = () => {
     state.isShow = false
   }
  
  const playMV = async mvid =>{
    const { data } = await axios.get(
      'https://autumnfish.cn/mv/url?id=' + mvid
    )
    state.isShow = true
    state.mvUrl=data.data.url;
  }

    const methods = { 
      searchMusic,
      playMusic,
      play,
      pause,
      hide,
      playMV
    }
    return{

       //data->{quary:''}
      //query:state.query
      // 转换数据toRefs() 
      ...toRefs(state),
      ...methods
    }

  }

}
</script>

<style lang="scss">
@import './css/index.css'
</style>
