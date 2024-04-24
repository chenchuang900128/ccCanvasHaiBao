# cc-videoSwiper

#### 使用方法 
```使用方法
<!-- goodsData：轮播图视频数据  @setShowVideo：视频按钮点击事件 -->
<cc-videoSwiper :goodsData="goodsData" @setShowVideo="setShowVideo"></cc-videoSwiper>
```

#### HTML代码实现部分
```html
<template>
	<view class="content">

		<!-- goodsData：轮播图视频数据  @setShowVideo：视频按钮点击事件 -->
		<cc-videoSwiper :goodsData="goodsData" @setShowVideo="setShowVideo"></cc-videoSwiper>

		<!-- 预览视频弹窗 -->
		<view class="mask" v-if="showVideo == true" @touchmove.stop.prevent="ondefault" @click="hideShow">
			<view class="close">
				<image src="/static/images/goods/close.png"></image>
			</view>
		</view>
		<view class="previewvideo" v-if="showVideo == true">
			<view class="videos">
				<video class="nowvideos" id="nowVideo" v-if="showVideo == true" :src="goodsData.videos"
					:autoplay="showVideo" :show-center-play-btn="true" :show-mute-btn="true"
					:show-fullscreen-btn="false"></video>
			</view>
		</view>
		<!-- 用来承载H5预览视频的 -->
		<view style="position: absolute;top: -999upx;left: -999upx;">
			<video ref="newVideo" id="newVideo" :src="goodsData.videos" :autoplay="showVideo"
				:show-center-play-btn="false" :show-mute-btn="true" :show-fullscreen-btn="false"
				@fullscreenchange="hideShow"></video>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {

				goodsData: {

					videos: 'http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4',
					imgList: [
						"https://cdn.pixabay.com/photo/2016/08/11/23/48/mountains-1587287_1280.jpg",
						'https://cdn.pixabay.com/photo/2016/11/14/04/45/elephant-1822636_1280.jpg',
						'https://cdn.pixabay.com/photo/2018/08/12/15/29/hintersee-3601004_1280.jpg',
						'https://cdn.pixabay.com/photo/2017/05/09/03/46/alberta-2297204_1280.jpg'
					],

				},

				showVideo: false,
				newVideo: null



			}
		},
		onLoad() {

			this.newVideo = uni.createVideoContext('newVideo');

		},

		methods: {

			//操作视频
			setShowVideo(showVideo, isH5) {
				this.showVideo = showVideo
				if (isH5 == true) {
					this.newVideo.play()
				}
				console.log('视频点击播放');
			},
			// 关闭视频
			hideShow() {
				this.showVideo = false
			},

		}
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;

	}

	/* 预览视频弹窗 */
	.mask {
		width: 100%;
		height: 100vh;
		position: fixed;
		top: 0;
		left: 0;
		background-color: rgba(0, 0, 0, .8);
		z-index: 200;
	}

	.previewvideo {
		width: 100vw;
		height: 100vw;
		position: fixed;
		top: 50%;
		left: 0;
		transform: translateY(-50%);
		background-color: #000;
		z-index: 900;
		opacity: 1;
	}

	.close {
		display: flex;
		align-content: center;
		align-items: flex-end;
		position: absolute;
		top: 140upx;
		right: 20upx;
		z-index: 900;

		image {
			width: 50upx;
			height: 50upx;
			display: block;
			justify-content: center;
			margin-left: 30upx;
			margin-bottom: 20upx;
			border-radius: 50%;
			padding: 10upx;
			background-color: rgba(0, 0, 0, 0.2);
		}
	}

	.videos {
		height: 100vw;
		width: 100vw;
		z-index: 10;
		position: relative;

		video {
			width: 100%;
			height: 100%;
		}
	}

	.nowvideos {
		width: 100%;
		height: 100%;
		margin: 0 auto;
	}
</style>



```