<template>
<div id="music-play">
	<md-theme :md-name="theme">
		<headerBack :title="activeSong.albumName"></headerBack>
	</md-theme> 
	<div class="container-wrap">
		<div class="container">
			<div class="img-wrap">
				<img :src="activeSong.avatarUrl">
			</div>
			<div class="main">
				<h2>{{activeSong.name}}</h2>
				<p>{{activeSong.singer}}</p>
				<div class="progress-wrap">
					<div class="time play">{{curTime}}</div>
					<div class="progress-bg">
						<div class="progress" :style="{width: activePercent}">
							<div class="progress-bar"></div>
						</div>
					</div>
					<div class="time total">{{activeSong.duration | formatTime}}</div>
				</div>
				<div class="panel">
					<i class="btn prev iconfont icon-prev" @click.prevent="playPrev()"></i>
					<i :class="['btn play iconfont', iconState]" @click.prevent="togglePlay()"></i>
					<i class="btn next iconfont icon-next" @click.prevent="playNext()"></i>
				</div>
			</div>
		</div>
	</div>
</div>
	
</template>
<script>
import Search from "./search_music.vue"
import BackToTop from "../common/backToTop.vue"
import axios from "axios"
import Store from "../../assets/js/storage.js"
import HeaderBack from "../common/header_back.vue"
export default {
  components: {
  	search: Search,
  	backToTop: BackToTop,
  	headerBack: HeaderBack
  },
  data() {
	return {
		id: this.$route.params.id,
	};
  },
  computed:{
    theme(){
    	return this.$store.getters.THEME_COLOR
    },
    playList(){
    	return this.$store.state.music.activeList || Store.get("music_list_"+ Store.get('activeListId'))
    },
    activeIndex(){
	   	return this.$store.state.music.activeSong.activeIndex;
	},
    activeSong(){
     // 从本地获取歌曲数据
	    /* var activeSongLocal = Store.get('activeSong');
	   	 if(typeof activeSongLocal !== "undefined" && typeof activeSongLocal !== null){
	   	 	return{
		     	id: this.id,
				name: activeSongLocal.name,
				singer: activeSongLocal.singer,
				duration: activeSongLocal.duration
	     	}
	   	 }*/
 	 // 从vux获取歌曲数据
     var lists = this.playList;
     var index = this.$store.state.music.activeSong.activeIndex || Store.get('activeSong').activeIndex;
     return {
     	id: lists && lists[index] && lists[index].id || 0,
     	name: lists && lists[index] && lists[index].name || '',
		singer: lists && lists[index] && lists[index].artists && lists[index].artists[0].name || '',
		duration: this.$store.state.music.activeSong.duration,
		avatarUrl: lists && lists[index] && lists[index].album.picUrl || '',
		albumName: lists && lists[index] && "专辑："+lists[index].album.name || ''
     }
   },
   playState(){
   	return this.$store.state.music.playing
   },
   iconState(){
   	return this.playState === true ? "icon-pause" : "icon-play"
   },
   curTime(){
      return this.$store.state.music.activeTime
   },
   activePercent(){
   	return this.$store.state.music.activePercent
   }
  },
  mounted:function(){
  	console.log(this.activeSong)
  	/*axios.get(API_PROXY+'http://music.163.com/api/song/lyric?os=pc&lv=-1&kv=-1&tv=-1&id='+this.id)
	  .then(function (res) {
	  	console.log(res);

	  }.bind(this))
	  .catch(function (error) {
	    console.log(error);
	  });*/
  },
  filters:{
	getShortName(val){
		if(String(val.length) > 10)
			return val.slice(0, 10)+"...";
		return val;
  	},
  	formatTime(val){
  		var m = Math.floor(val/1000/60).toString();
  		var s = Math.round(val/1000%60).toString();
  		m = (m.length == 1) ? 0+m : m;
  		s = (s.length == 1) ? 0+s : s;
  		return m+":"+s;
  	}
  },
  methods: {
   togglePlay(){
   	 if(this.playState){
   	 	this.$store.commit("PLAY_STATE_CHANGE",{
   	 		'playing': false
   	 	})	
   	 }else{
   	 	this.$store.commit("PLAY_STATE_CHANGE",{
   	 		'playing': true
   	 	})
   	 }
   },
   playNext(){
   		var index = ++this.activeIndex;
   		index = index < 0 ? 0 : index;
   		index = index > this.playList.length-1 ? this.playList.length-1 : index;
   		console.log(this.playList[index].artists[0].picUrl)
   		/*提交MUSIC_SONG_CHANGE的mutation*/
    	this.$store.commit('MUSIC_SONG_CHANGE',{
    		avatarUrl: this.playList[index].album.picUrl,
    		activeSrc: this.playList[index].mp3Url,
    		activeIndex: index,
    		duration: this.playList[index].duration,
    		playing: true
    	})
    	/*更改视图歌曲信息*/
    	this.activeSong = {
    		id: this.playList[index].id,
			name: this.playList[index].name,
			singer: this.playList[index].artists[0].name,
			avatarUrl: this.playList[index].album.picUrl,
			duration: this.playList[index].duration,
			activeIndex: index
    	}
    	/*更改本地歌曲信息*/
    	Store.set('activeSong',{
    		id: this.playList[index].id,
			name: this.playList[index].name,
			singer: this.playList[index].artists[0].name,
			avatarUrl: this.playList[index].album.picUrl,
			duration: this.playList[index].duration,
			activeIndex: index
    	});
   },
   playPrev(){
   		var index = --this.activeIndex;
   		index = index < 0 ? 0 : index;
   		index = index > this.playList.length-1 ? this.playList.length-1 : index;
   		/*提交MUSIC_SONG_CHANGE的mutation*/
    	this.$store.commit('MUSIC_SONG_CHANGE',{
    		avatarUrl: this.playList[index].album.picUrl,
    		activeSrc: this.playList[index].mp3Url,
    		activeIndex: index,
    		duration: this.playList[index].duration,
    		playing: true
    	})
    	/*更改视图歌曲信息*/
    	this.activeSong = {
    		id: this.playList[index].id,
			name: this.playList[index].name,
			singer: this.playList[index].artists[0].name,
			avatarUrl: this.playList[index].album.picUrl,
			duration: this.playList[index].duration,
			activeIndex: index
    	}
    	/*更改本地歌曲信息*/
    	Store.set('activeSong',{
    		id: this.playList[index].id,
			name: this.playList[index].name,
			singer: this.playList[index].artists[0].name,
			avatarUrl: this.playList[index].album.picUrl,
			duration: this.playList[index].duration,
			activeIndex: index
    	});
   }


  }
}
</script>
<style scoped lang="scss">
    h2{
    	font-size: 1.6rem;
    }
	div,p{
		font-size: 1rem;
		color: #fff;
	}
	#music-play{
		min-height: 100vh;
		box-sizing: border-box;
		padding: 64px 0 0 0;
		text-align: center;
	} 
	.container-wrap{
		height: 100vh;
		box-sizing: border-box;
		padding-top: 64px;
		margin-top: -64px;
		display: flex;
		align-items: center;
		justify-content:center;
		background-color: #545863;
	}
	.container{
		width: 90%;
		height: 95%;
		box-shadow: 0px 0px 10px 0px #2a2c33;
		.img-wrap{
			position: relative;
			height: 60%;
			img{
				width: 100%;
				height: 100%;
				opacity: 0.5;
			}
			&::after{
				content: "";
				position: absolute;
				width: 100%;
				height: 100%;
				left:0;
			}
		}	
		.main{
			position: relative;
			height:40%;
			box-sizing: border-box;
			padding: 6% 0;
			h2{
				margin: 10px 0;
				line-height: 1.0;
			}
		}
	}
	
	.progress-wrap{
		margin: 4% 0;
		display: flex;
		align-items: center;
		.time{
			flex: 2
		}
		.progress-bg{
			position: relative;
			height: 4px;
			background-color: #757575;
			flex: 8;
			.progress{
				position: absolute;
				height:100%;
				left: 0;
				background-color: #fff;
				.progress-bar{
					position: absolute;
					width: 12px;
					height: 12px;
					border-radius: 50%;
					background-color: #fff;
					left: 100%;
					top:50%;
					margin-top: -6px;
				}
			}
		}
	}
	.panel{
		height: 60px;
		display: flex;
		align-items: center;
		.btn{
			color: #fff;
			font-size: 40px;
		}
		.prev,.next{
			flex: 4;
		}
		.play{
			font-size: 80px;
			flex: 2;
		}
	}
</style>



