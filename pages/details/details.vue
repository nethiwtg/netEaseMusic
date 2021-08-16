<template>
	<view class="detail">
		<view class="fixbg" style="background-image: url(../../static/1.jpg);"></view>
		<musichead :title='songDetail.name' :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<view class="detail-play" @click="handleToplay">
					<image :class="{'detail-play-run':isPlayRotate}"  :src="songDetail.al.picUrl" mode=""></image>
					<text class="iconfont" :class="iconPlay"></text>
				</view>
				<view class="detail-lyric">
					<view class="detail-lyric-wrap" :style="{transform : 'translateY('+ -(lyricIndex-1)*82+'rpx)'}">
						<!-- <view class="detail-lyric-item">测试文字测试文字</view> -->
						<view class="detail-lyric-item" :class="{active:lyricIndex==index}" v-for="(item,index) in songLyric" :key="index">{{item.lyric}}</view>
						
					</view>
				</view>
				<view class="detail-like">
					<view class="detail-like-head">喜欢这首歌的人也听</view>
					<view class="detail-like-item" v-for="(item,index) in songSimi" :key="index" @click="handleToSimi(item.id)">
						<view class="detail-like-img">
							<image :src="item.album.picUrl" mode=""></image>
						</view>
						<view class="detail-like-song">
							<view>{{item.name}}</view>
							<view>
								<image v-show="item.privilege.flag>60&&item.privilege.flag<70" src="../../static/dujia.svg" mode=""></image>
								<image v-show="item.privilege.maxbr==999000" src="../../static/2-1-3quality.svg" mode=""></image>
								{{item.album.artists[0].name}}- {{item.name}}
							</view>
						</view>
						<text class="iconfont icon-bofang1 "></text>
					</view>
					
					
				</view>
				<view class="detail-comment">
					<view class="detail-comment-head">精彩评论</view>
					<view class="detail-comment-item" v-for="(item,index) in songComment" :key="index">
						<view class="detail-comment-img">
							<image :src="item.user.avatarUrl" mode=""></image>
						</view>
						<view class="detail-comment-content">
							<view class="detail-comment-title">
								<view class="detail-comment-name">
									<view>{{item.user.nickname}}</view>
									<view>{{item.time | formatTime}}</view>
								</view>
								<view class="detail-comment-like">{{item.likedCount | formatCount}} <text class="iconfont icon-dianzan"></text></view>
							</view>
							<view class="detail-comment-text">{{item.content}}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import '@/common/iconfont.css'
	import musichead from '../../components/musichead/musichead.vue'
	import {songDetail,songLyric,songSimi,songUrl,songComment} from '../../common/api.js'
	export default {
		data() {
			return {
				songDetail:{
					al:{
						picUrl:''
					}
				},
				songSimi:[],
				songComment:[],
				songLyric:[],
				lyricIndex:0,
				iconPlay: 'icon-ico_zanting_xuanzhong',
				isPlayRotate:true,
				isLoading :true
			}
		},
		components:{
			musichead
		},
		onUnload(){
			this.cancelLyricIndex()
			// #ifdef H5
			this.bgAudioManager.destroy()
			// #endif
			
		},
		// onHide(){
		// 	this.cancelLyricIndex()
		// 	// #ifdef H5
		// 	this.bgAudioManager.destroy()
		// 	// #endif
		// },
		onLoad(options){
			// console.log(options.songId)
			this.getMusic(options.songId)
			// songComment(options.songId).then((res)=>{
			// 	// console.log(res)
			// 	this.songComment=res[1].data.hotComments
			// })
			// songLyric(options.songId).then((res)=>{
			// 	// console.log(res)
			// 	let lyric =res[1].data.lrc.lyric
			// 	// console.log(lyric)
			// 	let re =/\[([^\]]+)\]([^\[]+)/g;
			// 	var result=[]
			// 	lyric.replace(re,($0,$1,$2)=>{
			// 		result.push({"time":this.formatTimetoSecond($1),"lyric":$2})
			// 	})
			// 	this.songLyric=result
			// })
			// songUrl(options.songId).then((res)=>{
			// 	if(res[1].data.code=='200'){
			// 		// console.log(res)
			// 		// #ifdef MP-WEIXIN
			// 		this.bgAudioManager = uni.getBackgroundAudioManager();
			// 		this.bgAudioManager.title = this.songDetail.name;
			// 		// #endif
					
			// 		// #ifdef H5
			// 		if(!this.bgAudioManager){
			// 			this.bgAudioManager=uni.createInnerAudioContext()
			// 		}
			// 		this.iconPlay='icon-bofang1'
			// 		this.isPlayRotate=false
			// 		// #endif
					
			// 		this.bgAudioManager.src =res[1].data.data[0].url || '';
			// 		this.listenLyricIndex()
			// 		this.bgAudioManager.onPlay(()=>{
			// 			this.iconPlay='icon-ico_zanting_xuanzhong';
			// 			this.isPlayRotate=true
			// 			this.listenLyricIndex()
			// 		})
			// 		this.bgAudioManager.onPause(()=>{
			// 			this.iconPlay='icon-bofang1'
			// 			this.isPlayRotate=false
			// 			this.cancelLyricIndex()
			// 		})
			// 	}
			// })
		},
		methods: {
			getMusic(songId){
				this.$store.commit('NEXT_ID',songId)
				uni.showLoading({
					title:'加载中...'
				})
				this.isLoading=true
				Promise.all([songDetail(songId),songSimi(songId),songComment(songId),songLyric(songId),songUrl(songId)]).then((res)=>{
					// console.log(res)
					if(res[0][1].data.code==200){
						
						this.songDetail=res[0][1].data.songs[0]
					}
					if(res[1][1].data.code=='200'){
						// console.log(res)
						this.songSimi=res[1][1].data.songs
					}
					if(res[2][1].data.code=='200'){
						this.songComment=res[2][1].data.hotComments
					}
					if(res[3][1].data.code=='200'){
						let lyric =res[3][1].data.lrc.lyric
							// console.log(lyric)
							let re =/\[([^\]]+)\]([^\[]+)/g;
							var result=[]
							lyric.replace(re,($0,$1,$2)=>{
								result.push({"time":this.formatTimetoSecond($1),"lyric":$2})
							})
							this.songLyric=result
					}
					if(res[4][1].data.code=='200'){
						// #ifdef MP-WEIXIN
						this.bgAudioManager = uni.getBackgroundAudioManager();
						this.bgAudioManager.title = this.songDetail.name;
						// #endif
						
						// #ifdef H5
						if(!this.bgAudioManager){
							this.bgAudioManager=uni.createInnerAudioContext()
						}
						this.iconPlay='icon-bofang1'
						this.isPlayRotate=false
						// #endif
						
						this.bgAudioManager.src =res[4][1].data.data[0].url || '';
						this.listenLyricIndex()
						this.bgAudioManager.onPlay(()=>{
							this.iconPlay='icon-ico_zanting_xuanzhong';
							this.isPlayRotate=true
							this.listenLyricIndex()
						})
						this.bgAudioManager.onPause(()=>{
							this.iconPlay='icon-bofang1'
							this.isPlayRotate=false
							this.cancelLyricIndex()
						})
						this.bgAudioManager.onEnded(()=>{
							this.getMusic(this.$store.state.nextId)
							this.bgAudioManager.autoplay=true
						})
					}
					this.isLoading=false
					uni.hideLoading()
				})
			},
			formatTimetoSecond(value){
				let arr=value.split(':')
				return  (Number(arr[0]*60)+Number(arr[1])).toFixed(1)
			},
			handleToplay(){
				if(this.bgAudioManager.paused){
					this.bgAudioManager.play()
				}
				else{
					this.bgAudioManager.pause()
				}
			},
			listenLyricIndex(){
				clearInterval(this.timer)
				this.timer=setInterval(()=>{
					for(var i=0;i<this.songLyric.length;i++){
						if(this.bgAudioManager.currentTime>this.songLyric[this.songLyric.length-1].time){
							this.lyricIndex=this.songLyric.length-1
							break
						}
						if(this.bgAudioManager.currentTime>this.songLyric[i].time && this.bgAudioManager.currentTime<this.songLyric[i+1].time){
							this.lyricIndex=i
						}
					}
					console.log(this.lyricIndex)
				},500)
			},
			cancelLyricIndex(){
				clearInterval(this.timer)
			},
			handleToSimi(songId){
				this.getMusic(songId)
				songComment(songId)
				songLyric(songId)
				songUrl(songId)
			}
		}
	}
</script>

<style lang="less" scoped>
	.detail-play{
		width: 580rpx;
		height: 580rpx;
		background:url(~@/static/changpian.png);
		background-size: cover;
		margin: 200rpx auto 0 auto;
		position: relative;
		
	}
	.detail-play image{
		width: 310rpx;
		height: 310rpx;
		border-radius: 50%;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		margin: auto;
		margin-top: 160rpx;
		animation: 10s linear move infinite;
		animation-play-state: paused;
	}
	
	.detail-play .detail-play-run{
		animation-play-state:running ;
	}
	@keyframes move{
		from{transform: rotate(0deg);}
		to{transform: rotate(360deg);}
	}
	.detail-play text{
		height: 100rpx;
		width: 100rpx;
		font-size: 100rpx;
		color: white;
		
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		margin: auto;
		opacity: 0.5;
	}
	.detail-lyric{
		font-size: 32rpx;
		line-height: 82rpx;
		height: 246rpx;
		text-align: center;
		overflow: hidden;
		color: #6f6e37;
	}
	.detail-lyric-wrap{
		transition: .5s;
	}
	.detail-lyric-item{height: 82rpx;}
	.detail-lyric-item.active{
		color: white;
	}
	
	.detail-like{margin: 0 30rpx;}
	.detail-like-head{
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
		
	}
	.detail-like-item{
		display: flex;
		align-items: center;
		margin-bottom: 28rpx;
	}
	.detail-like-item text{font-size: 50rpx;color: #c6c2bf;}
	.detail-like-img{
		width: 82rpx;
		height: 82rpx;
		border-radius:20rpx ;
		overflow: hidden;
		margin-right: 20rpx;
	}
	.detail-like-img image{
		width: 100%;
		height: 100%;
	} 
	.detail-like-song view:nth-child(1){
		font-size: 28rpx;
		color: white;
		margin-bottom: 12rpx;
	}
	.detail-like-song view:nth-child(2){
		font-size: 22rpx;
	}
	.detail-like-song{flex: 1;color:(#c6c2bf)}
	.detail-like-song image{
		width: 26rpx;
		height: 20rpx;
		margin-right: 10rpx;
	}
	.detail-comment{margin: 0 30rpx;}
	.detail-comment-head{font-size: 36rpx; color: white; margin: 50rpx; margin-left: 0;}
	.detail-comment-item{margin-bottom: 28rpx;
	display: flex;
	}
	.detail-comment-img{
		width: 64rpx;
		height: 64rpx;
		border-radius: 50%;
		overflow: hidden;
		margin-right: 18rpx;
	}
	.detail-comment-img image{
		width: 100%;
		height: 100%;
	}
	.detail-comment-content{
		flex: 1;
		color: #cbcacf;
	}
	.detail-comment-title{
		display: flex;
		justify-content:space-between ;
	}
	.detail-comment-name{}
	.detail-comment-name view:nth-child(1){
		font-size: 26rpx;
	}
	.detail-comment-name view:nth-child(2){
		font-size: 20rpx;
	}
	.detail-comment-like{font-size: 28rpx;}
	.detail-comment-text{
		font-size: 28rpx;
		line-height: 40rpx;
		color: white;
		margin-top: 20rpx;
		border-bottom: 1px #e0e0e0 solid;
		padding-bottom: 40rpx;
	}
</style>
