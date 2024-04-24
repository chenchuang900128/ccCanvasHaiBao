<template>
	<view>
		<!-- 绘制canvas -->
		<canvas class="mycanvas" canvas-id="mycanvas" :style="'width:' + (windowWidth-60) + 'px;height:520px'"></canvas>

		<block>
			<!-- 展示图片 -->
			<image class="imgs" :style="'width:' + (windowWidth-60) + 'px;height:520px'" :src="imgUrl" mode=""></image>

		</block>


		<!-- #ifndef H5 -->
		<view class="saveImg" @tap="saveImg" :style="'background:' + colors">
			保存图片
		</view>
		<!-- #endif -->
		<!-- #ifdef H5 -->
		<p class="tips">长按图片进行保存</p>
		<!-- #endif -->
	</view>
</template>

<script>
	import uQRCode from './uqrcode.js'

	export default {
		mixins: [{
			methods: {
				setData: function(obj, callback) {
					let that = this;
					const handleData = (tepData, tepKey, afterKey) => {
						tepKey = tepKey.split('.');
						tepKey.forEach(item => {
							if (tepData[item] === null || tepData[item] === undefined) {
								let reg = /^[0-9]+$/;
								tepData[item] = reg.test(afterKey) ? [] : {};
								tepData = tepData[item];
							} else {
								tepData = tepData[item];
							}
						});
						return tepData;
					};
					const isFn = function(value) {
						return typeof value == 'function' || false;
					};
					Object.keys(obj).forEach(function(key) {
						let val = obj[key];
						key = key.replace(/\]/g, '').replace(/\[/g, '.');
						let front, after;
						let index_after = key.lastIndexOf('.');
						if (index_after != -1) {
							after = key.slice(index_after + 1);
							front = handleData(that, key.slice(0, index_after), after);
						} else {
							after = key;
							front = that;
						}
						if (front.$data && front.$data[after] === undefined) {
							Object.defineProperty(front, after, {
								get() {
									return front.$data[after];
								},
								set(newValue) {
									front.$data[after] = newValue;
									that.$forceUpdate();
								},
								enumerable: true,
								configurable: true
							});
							front[after] = val;
						} else {
							that.$set(front, after, val);
						}
					});
					isFn(callback) && this.$nextTick(callback);
				}
			}
		}],
		data() {
			return {
				windowWidth: '',
				windowHeight: '',
				colors: "#fa436a",
				ctx: '',
				imgUrl: '',
				isShow: false,

			};
		},


		props: {
			// 海报生成数据
			posterData: {
				type: Object
			}
		},
		created() {
			this.getSystem();

			this.setPoster();
		},
		/**
		 * 页面上拉触底事件的处理函数
		 */
		onReachBottom: function() {},

		/**
		 * 用户点击右上角分享
		 */
		onShareAppMessage: function() {},
		methods: {
			getSystem() {
				let that = this;
				uni.getSystemInfo({
					success: function(res) {
						console.log(res);
						that.setData({
							windowHeight: res.windowHeight,
							windowWidth: res.windowWidth
						});
					}
				});
			},

			setPoster() {
				uni.showLoading({
					title: '海报生成中...'
				})
				let ctx = uni.createCanvasContext('mycanvas', this); // 绘制背景
				ctx.fillStyle = "#FFFFFF"
				ctx.fillRect(0, 0, this.windowWidth - 60, 520)
				/**
				 * 绘制名称
				 */
				const setText = (context, fs, color, x, y, c, bold) => {
					context.setFillStyle(color);
					context.setTextAlign('left');
					if (bold) {
						context.font = 'normal bold 20px Arial,sans-serif';
					} else {
						context.font = 'normal 20px Arial,sans-serif';
					}
					context.setFontSize(fs);
					context.fillText(c, x, y);
					context.restore();
				};
				setText(ctx, 14, '#333', 85, 35, this.posterData.name, 'bold');
				setText(ctx, 12, '#999', 85, 58, '为您挑选了一个好礼物');
				ctx.save();
				/**
				 * 绘制头像
				 */
				ctx.beginPath();
				let avatar_width = 60; //头像宽度
				let avatar_height = 60; //头像高度
				let avatar_x = 15; //头像的x坐标
				let avatar_y = 15; //头像的y坐标
				let radius = 8 //头像的圆角弧度
				// 绘制圆角头像
				this.setRadius(ctx, avatar_width, avatar_height, avatar_x, avatar_y, radius)
				// 绘制圆形图片
				// this.setCircular(ctx, avatar_width, avatar_height, avatar_x, avatar_y)

				// 绘制商品图片
				this.setGoodsImg(ctx)

				// 绘制商品价格
				let pirce = '￥ ' + this.posterData.money
				this.setGoodsPrice(ctx, 20, this.colors, 15, 410, pirce)
				// 绘制商品名称
				this.setGoodsName(ctx)

			},

			setEwm(ctx) {


				console.log('生成二维码')
				let code_widht = 100; //二维码宽度
				let code_height = 100; //二维码高度
				let x = this.windowWidth - 170


				let myThis = this;
				console.log(this.windowWidth);
			
				
				uQRCode.make({
					canvasId: 'mycanvas',
					componentInstance: this,
					text: this.posterData.url,
					size: 200,
					margin: 10,
					fileType: 'png',
					success: res => {
						console.log('生成二维码3')

						console.log('res = ' + res);

						ctx.drawImage(res, x, 410, code_widht, code_height);
					
						setTimeout(() => { //必须延时执行 不然h5不显示
							ctx.save();
							ctx.draw(false, () => {
								setTimeout(() => {
									uni.canvasToTempFilePath({
										canvasId: 'mycanvas',
										success: (res) => {
											console.log(res)
											this.imgUrl = res.tempFilePath
										}
									}, this)
								}, 300)
							})
							uni.hideLoading()
						}, 1000)

					},
					complete: () => {

					}
				})


			},
			setRadius(ctx, avatar_width, avatar_height, avatar_x, avatar_y, radius) {
				/**
				 * 绘制圆角
				 */
				ctx.arc(avatar_x + radius, avatar_y + radius, radius, Math.PI, Math.PI * 3 / 2);
				ctx.lineTo(avatar_width - radius + avatar_x, avatar_y);
				ctx.arc(avatar_width - radius + avatar_x, radius + avatar_y, radius, Math.PI * 3 / 2, Math.PI * 2);
				ctx.lineTo(avatar_width + avatar_x, avatar_height + avatar_y - radius);
				ctx.arc(avatar_width - radius + avatar_x, avatar_height - radius + avatar_y, radius, 0, Math.PI * 1 / 2);
				ctx.lineTo(radius + avatar_x, avatar_height + avatar_y);
				ctx.arc(radius + avatar_x, avatar_height - radius + avatar_y, radius, Math.PI * 1 / 2, Math.PI);
				// 开始填充
				ctx.strokeStyle = "#fff";
				ctx.fill() //保证图片无bug填充
				ctx.clip(); //画了圆 再剪切  原始画布中剪切任意形状和尺寸。一旦剪切了某个区域，则所有之后的绘图都会被限制在被剪切的区域内

				ctx.drawImage(this.posterData.logo, avatar_x, avatar_y, avatar_width, avatar_height);
				ctx.closePath()
				ctx.restore();
			},
			setCircular(ctx, avatar_width, avatar_height, avatar_x, avatar_y) { //绘制圆形图片
				/**
				 * 绘制圆形
				 */
				//先画个圆   前两个参数确定了圆心 （x,y） 坐标  第三个参数是圆的半径  四参数是绘图方向  默认是false，即顺时针
				ctx.arc(avatar_width / 2 + avatar_x, avatar_height / 2 + avatar_y, avatar_width / 2, 0, Math.PI * 2,
					false); //画圆
				// 开始填充
				ctx.strokeStyle = "#fff";
				ctx.fill() //保证图片无bug填充
				ctx.clip(); //画了圆 再剪切  原始画布中剪切任意形状和尺寸。一旦剪切了某个区域，则所有之后的绘图都会被限制在被剪切的区域内

				ctx.drawImage(this.posterData.logo, avatar_x, avatar_y, avatar_width, avatar_height);
				ctx.closePath()
				ctx.restore();
			},
			setGoodsImg(ctx) { //绘制中间商品图片
				let width = this.windowWidth - 90
				ctx.drawImage(this.posterData.img, 15, 95, width, width);
				ctx.save();
			},
			setGoodsPrice(ctx, fs, color, x, y, c, bold) { //绘制商品价格
				ctx.setFillStyle(color);
				ctx.setTextAlign('left');
				if (bold) {
					ctx.font = 'normal bold 20px Arial,sans-serif';
				} else {
					ctx.font = 'normal 20px Arial,sans-serif';
				}
				ctx.setFontSize(fs);
				ctx.fillText(c, x, y);
				ctx.restore();
			},
			setGoodsName(ctx) { //绘制商品名称
				let obj = {
					x: 20, //绘制文本的左上角x坐标位置
					y: 440, //绘制文本的左上角y坐标位置
					width: 210,
					height: 20,
					line: 2,
					color: '#202020',
					size: 14, //字体的字号
					align: 'left',
					baseline: 'top',
					text: this.posterData.title,
					bold: true
				};
				var td = Math.ceil(obj.width / (obj.size));
				var tr = Math.ceil(obj.text.length / td);
				for (var i = 0; i < tr; i++) {
					var txt = {
						x: obj.x,
						y: obj.y + (i * obj.height),
						color: obj.color,
						size: obj.size,
						align: obj.align,
						baseline: obj.baseline,
						text: obj.text.substring(i * td, (i + 1) * td),
						bold: obj.bold
					};
					if (i < obj.line) {
						if (i == obj.line - 1) {
							txt.text = txt.text.substring(0, txt.text.length - 3) + '......';
						}
						this.drawText(ctx, txt);
					}
				}
				// 绘制二维码
				this.setEwm(ctx);
			},
			/**
			 * 渲染文字
			 *
			 * @param {Object} obj
			 */
			drawText: function(ctx, obj) {
				console.log('渲染文字', obj)
				ctx.save();
				ctx.setFillStyle(obj.color);
				ctx.setFontSize(obj.size);
				ctx.setTextAlign(obj.align);
				ctx.setTextBaseline(obj.baseline);
				if (obj.bold) {
					console.log('字体加粗')
					ctx.fillText(obj.text, obj.x, obj.y - 0.1);
					ctx.fillText(obj.text, obj.x - 0.1, obj.y);
				}
				ctx.fillText(obj.text, obj.x, obj.y);
				if (obj.bold) {
					ctx.fillText(obj.text, obj.x, obj.y + 0.1);
					ctx.fillText(obj.text, obj.x + 0.1, obj.y);
				}
				ctx.restore();
			},
			saveImg() {
				//保存图片
				uni.showLoading({
					title: '保存中...'
				});
				let that = this;
				// #ifdef MP
				uni.getSetting({
					success(res) {
						uni.hideLoading();
						console.log(res)
						if (!res.authSetting['scope.writePhotosAlbum']) {
							wx.authorize({
								scope: 'scope.writePhotosAlbum',
								success: (res) => {
									console.log(res)
								}
							})
						} else {
							setTimeout(() => {
								uni.saveImageToPhotosAlbum({
									filePath: that.imgUrl,
									success(re) {
										uni.hideLoading();
										uni.showToast({
											title: '保存成功',
											icon: 'success'
										});
									},
									fail(err) {
										console.log(err);
										uni.showToast({
											title: '保存失败',
											icon: 'none'
										});
									}

								});
							}, 1000);
						}
					}

				});
				// #endif
				// #ifdef APP-PLUS
				setTimeout(() => {
					uni.saveImageToPhotosAlbum({
						filePath: that.imgUrl,
						success(re) {
							uni.hideLoading();
							uni.showToast({
								title: '保存成功',
								icon: 'success'
							});
						},
						fail(err) {
							console.log(err);
							uni.showToast({
								title: '保存失败',
								icon: 'none'
							});
						}

					});
				}, 1000);
				// #endif
			}
		}
	};
</script>
<style scoped>
	.mycanvas {
		margin: 0 auto;
		border-radius: 10upx;
		overflow: hidden;
	}

	.saveImg {
		width: 70%;
		height: 80upx;
		line-height: 80upx;
		text-align: center;
		color: #fff;
		background-color: #4DB8E4;
		border-radius: 10upx;
		margin: 40upx auto 20upx;
	}

	.imgs {
		position: absolute;
		top: 0;
		left: 50%;
		border-radius: 10upx;
		transform: translateX(-50%);
		/* 遮挡二维码 */
		background-color: white;
		/* 置于上层 */
		z-index: 99;
	}

	.tips {
		text-align: center;
		color: #fff;
		font-size: 24upx;
		margin-top: 20upx;
	}
</style>