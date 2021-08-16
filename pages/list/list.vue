<template>
	<view class="list">
		<view class="fixbg" style="background-image: url(../../static/2.jpg);"></view>
		<musichead title='歌单' :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<view class="list-head">
					<view class="list-head-img">
						<image :src="playlist.coverImgUrl" mode=""></image>
						<text class="iconfont icon-yousanjiao">{{playlist.playCount | formatCount}}</text>
					</view>
					<view class="list-head-text">
						<view>{{playlist.name}}</view>
						<view>
							<image :src="playlist.creator.avatarUrl" mode=""></image>{{playlist.creator.nickname}}
						</view>
						<view>{{playlist.description}}</view>
					</view>
				</view>
				<button class="list-share" open-type="share">
					<text class="iconfont icon-fenxiang"></text>分享给微信好友
				</button>
				<view class="list-music">
					<view class="list-music-head">
						<text class="iconfont icon-bofang"></text>
						<text>播放全部</text>
						<text>(共{{playlist.trackCount}}首)</text>
					</view>
					<view class="list-music-item" @click="handleToDetail(item.id)" v-for="(item,index) in playlist.tracks" :key="index">
						<view class="list-music-top">{{index+1}}</view>
						<view class="list-music-song">
							<view>{{item.name}}</view>
							<view>
								<image v-if="privileges[index].flag>60 && privileges[index].flag<70 " src="../../static/dujia.svg" mode=""></image>
								<image v-if="privileges[index].maxbr==999000" src="../../static/2-1-3quality.svg" mode=""></image>
								{{item.ar[0].name}} - {{item.name}}
							</view>
						</view>
						<text class="iconfont icon-bofang1"></text>
					</view>
					
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import '@/common/iconfont.css'
	import musichead from '../../components/musichead/musichead.vue'
	import {list} from '../../common/api.js'
	export default {
		data() {
			return {
				playlist:{
					creator:{
						nickname:''
					},
					coverImgUrl:''
				},
				privileges:[],
				isLoading:true
			}
		},
		components: {
			musichead
		},
		onLoad(options) {
			uni.showLoading({
				title:'加载中...'
			})
			// console.log(options.listId)
			list(options.listId).then((res)=>{
				if(res[1].data.code=='200'){
					console.log(res)
					this.playlist=res[1].data.playlist
					this.privileges=res[1].data.privileges
					this.$store.commit('INIT_TOPLISTIDS',res[1].data.playlist.trackIds)
					this.isLoading=false
					uni.hideLoading()
				}
			})
		},
		methods: {
			handleToDetail(songId){
				uni.navigateTo({
					url: '/pages/details/details?songId='+songId,
				});
			}
		}
	}
</script>

<style lang="less" scoped>
	.list-head {
		display: flex;
		margin: 30rpx;
	}

	.list-head-img {
		width: 264rpx;
		height: 264rpx;
		border-radius: 30rpx;
		overflow: hidden;
		position: relative;
		margin-right: 42rpx;

		image {
			width: 100%;
			height: 100%;
		}

		text {
			position: absolute;
			right: 8rpx;
			top: 8rpx;
			color: white;
			font-size: 26rpx;
		}
	}

	.list-head-text {
		flex: 1;
		color: #f0f2f7;
	}

	.list-head-text view:nth-child(1) {
		color: black;
		font-size: 32rpx;
	}

	.list-head-text view:nth-child(2) {
		align-items: center;
		display: flex;
		margin: 20rpx 0;
		font-size: 24rpx;
		color: black;
	}

	.list-head-text view:nth-child(2) image {
		width: 54rpx;
		height: 54rpx;
		border-radius: 50%;
		margin-right: 14rpx;
	}

	.list-head-text view:nth-child(3) {
		line-height: 32rpx;
		font-size: 22rpx;
		color: black;
	}
	.list-share{
		width: 330rpx;
		height: 74rpx;
		margin: 0 auto;
		background: rgba(0,0,0,0.4);
		border-radius: 37rpx;
		color: white;
		text-align: center;
		line-height: 74rpx;
		font-size: 28rpx;
		text{
			margin-right: 16rpx;
		}
	}
	
	.list-music{
		// background: rgba(white,0.1);
		border-radius:50rpx;
		margin-top: 40rpx;
		overflow: hidden;
	}
	.list-music-head{
		height: 50rpx;
		margin: 30rpx 0 70rpx 22rpx;
	}
	.list-music-head text:nth-child(1){
		height: 50rpx;
		font-size: 50rpx;
		color: #c7c7c7;
	}
	.list-music-head text:nth-child(2){
		font-size: 30rpx;
		margin: 0 10rpx 0 26rpx;
		color: #c7c7c7;
	}
	.list-music-head text:nth-child(3){
		font-size: 26rpx;
		color: #c7c7c7;
	}
	.list-music-item{
		display: flex;
		margin: 0 32rpx 66rpx 46rpx;
		align-items: center;
		color: #c7c7c7;
	}
	.list-music-top{
		width: 58rpx;
		font-size: 30rpx;
		line-height: 30rpx;
	}
	.list-music-song{
		flex: 1;
	}
	.list-music-song view:nth-child(1){
		font-size: 28rpx;
		color: #f8f8f8;
		width: 70vw;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	.list-music-song view:nth-child(2){
		display: flex;
		font-size: 20rpx;
		align-items: center;
		margin-right: 10rpx;
		width: 70vw;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		}
	.list-music-song view:nth-child(2) image{
		width: 32rpx;
		height: 20rpx;
	}
	.list-music-item text{
		font-size: 50rpx;
		color: #c7c7c7;
	}
</style>
