# cc-beautyPoster

#### 使用方法 
```使用方法

<!-- posterData： 海报数据 -->
<cc-beautyPoster :posterData="posterData"></cc-beautyPoster>

<!-- 海报数据字段 注意小程序下网络图片地址需要是服务器设置的白名单地址 本地图片无限制！ -->
posterData: {
			// 用户姓名
			name: '小明',
			// 用户头像
			logo: '/static/images/headIcon.jpg',
			// 商品名称
			title: '精美时尚苹果手机一部',
			// 商品价格
			money: '5200.90',
			// 商品图片(小程序需要换成自己服务器白名单设置的地址)
			img: '/static/images/goods.jpg',
			// 商品链接
			url: 'https://www.apple.com.cn/iphone/'
			

	},			
				
```

#### HTML代码实现部分
```html
<template>
	<view class="content">

		<button style="margin-top: 38px;" @click="openPoster">生成商品海报</button>

		<!-- 海报弹框 -->
		<uni-popup ref="popup" type="center">
			<view class="center_poter" style="margin: 0 auto;" v-if="shows">
				<!-- #ifndef MP -->
				<image class="close_btn" src="/static/images/goods/close.png" @click="hidePoster">
				</image>
				<!-- #endif -->
				<!-- #ifdef MP -->
				<cover-image class="close_btn" src="/static/images/goods/close.png" @click="hidePoster">
				</cover-image>
				<!-- #endif -->

				<!-- posterData： 海报数据 -->
				<cc-beautyPoster :posterData="posterData"></cc-beautyPoster>
			</view>
		</uni-popup>

	</view>

</template>

<script>
	import uniPopup from '@/components/uni-popup/uni-popup.vue'

	export default {
		components: {
			uniPopup
		},
		data() {
			return {
				shows: false,
				posterData: {
					// 用户姓名
					name: '小明',
					// 用户头像
					logo: '/static/images/headIcon.jpg',
					// 商品名称
					title: '精美时尚苹果手机一部',
					// 商品价格
					money: '5200.90',
					// 商品图片(小程序需要换成自己服务器白名单设置的地址)
					img: '/static/images/goods.jpg',
					// 商品链接
					url: 'https://www.apple.com.cn/iphone/'
					
				},


			}
		},


		methods: {
			//生成海报
			openPoster() {
				this.shows = false
				uni.showLoading({
					title: '海报绘制中..'
				})
				this.$refs.popup.open()
				setTimeout(() => {
					uni.hideLoading()
					this.shows = true
				}, 400)
			},

			//关闭海报
			hidePoster() {
				this.$refs.popup.close()
			},

		}
	}
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;

	}

	.center_poter {
		position: relative;
		z-index: 999;

		.close_btn {
			width: 40upx;
			height: 40upx;
			background-color: rgba($color: #000000, $alpha: .3);
			position: absolute;
			top: 5upx;
			right: 5upx;
			z-index: 500;
			padding: 5upx;
			border-radius: 6upx;
			z-index: 999;
			text-align: center;
		}
	}
</style>
```