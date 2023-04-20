<template>
  <view class="ancientry">
		<view class="ancientry_original">
			<template v-if="originalImage">
				<image class="ancientry_original_image" @load="onImageLoad" :src="originalImage" mode="aspectFit" />
				<movable-area :style="{position: 'absolute', left:canvasLeft+'px', top:canvasTop+'px', width: canvasWidth+'px', height: canvasHeight+'px'}">
					<movable-view
						:x="x"
						:y="y"
						:scale="true"
						damping="900"
						friction="100"
						direction="all"
						bindscale="myscaleevent"
						bindchange="mychangeevent"
					></movable-view>
				</movable-area>
			</template>
			<template v-else>
				<view class="angle left-top-angle"></view>
				<view class="angle left-bottom-angle"></view>
				<view class="angle right-top-angle"></view>
				<view class="angle right-bottom-angle"></view>
			</template>
		</view>
		<view class="ancientry_original_operation">
			<view class="ancientry_original_button" @click="uploadImage">上传图片</view>
			<view class="ancientry_original_button" @click="removeWaterMark">生成古风</view>
		</view>
		<view class="ancientry_generated">
			<image :src="generatedImage" mode="aspectFit" />
		</view>
		<view class="ancientry_generated_operation">
			<view class="ancientry_generated_button" @click="getSaveImg">保存图片</view>
		</view>
	</view>
</template>

<style lang="scss" scoped>
	.ancientry {
		width: 100vw;
		height: 100vh;
		background-color: #FFF;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		.ancientry_original, .ancientry_generated {
			flex: 1;
			width: 100%;
			margin: 0 auto;
			border-bottom: 1px solid #ddd;
			border-top: 1px solid #DDD;
			image {
				width: 100%;
				height: 100%;
			}
		}
		.ancientry_original {
			border: 1px solid green;
			position: relative;
			.angle {
				border-right: 8rpx solid #999;
				border-top: 8rpx solid #999;
				width: 60rpx;
				height: 60rpx;
			}
			.left-top-angle {
				position: absolute;
				left: 32rpx;
				top: 32rpx;
				transform: rotate(270deg);
			}
			.left-bottom-angle {
				position: absolute;
				left: 32rpx;
				bottom: 32rpx;
				transform: rotate(180deg);
			}
			.right-top-angle {
				position: absolute;
				right: 32rpx;
				top: 32rpx;
			}
			.right-bottom-angle {
				position: absolute;
				right: 32rpx;
				bottom: 32rpx;
				transform: rotate(90deg);
			}
			movable-area {
				height: 200px;
				width: 200px;
				overflow: hidden;
				border: 1px solid goldenrod;
				movable-view {
					  display: flex;
					  align-items: center;
					  justify-content: center;
					  height: 100rpx;
					  width: 100rpx;
					  // background: #1AAD19;
						border: 1px solid red;
					  color: #fff;
					}
			}
		}
		.ancientry_original_operation {
			height: 100rpx;
			width: 100%;
			display: flex;
			margin-top: 12rpx;
			.ancientry_original_button {
				flex: 1;
				flex-direction: row;
				height: 64rpx;
				text-align: center;
				line-height: 64rpx;
				font-size: 26rpx;
				border: 1px solid #DDD;
				border-radius: 24rpx;
				&:nth-child(1) {
					margin-left: 32rpx;
					margin-right: 16rpx;
				}
				&:nth-child(2) {
					margin-right: 32rpx;
				}
			}
		}
		.ancientry_generated_operation {
			height: 88rpx;
			width: 100%;
			margin-top: 12rpx;
			.ancientry_generated_button {
				height: 64rpx;
				text-align: center;
				line-height: 64rpx;
				font-size: 26rpx;
				border: 1px solid #DDD;
				border-radius: 24rpx;
				margin: 0 32rpx;
			}
		}
	}
		
</style>

<script>
	import base64src from '@/utils/base64.js';
	export default {
		data() {
			return {
				imgWidth: 0, 								// 原始图片宽度
				imgHeight: 0, 							// 原始图片高度
				canvasLeft: 0, 						// 画布距离左边大小
				canvasTop: 0, 							// 画布距离顶部大小
				canvasWidth: 0, // 画布宽度
				canvasHeight: 0, // 画布高度
				originalWidth: 0, 					// 原始盒子宽度
				originalHeight: 0,					// 原始盒子高度
				originalImage: null, // 原始图片地址
				generatedImage: null, // 去水印后的图片
				qrcode: null,
			}
		},
		
		mounted() {
			// this.setPosition();
		},
		
		onReady() {
			// 获取上方边框的宽高
			let that = this;
			uni.createSelectorQuery()
					.select(".ancientry_original")
					.fields({ node: true, size: true })
					.exec(async (res) => {
						that.originalWidth = res[0].width;
						that.originalHeight = res[0].height;
					});
		},
			
		methods: {
			
			// 图片加载的时候触发
			onImageLoad() {
				let that = this;
				uni.createSelectorQuery()
						.select(".ancientry_original_image")
						.fields({ node: true, size: true })
						.exec(async (res) => {
							that.canvasWidth = res[0].width;
							that.canvasHeight = res[0].height;
						});
			},
			
			// 上传图片
			uploadImage() {
				let that = this;
				uni.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'], // 从相册选择
					success(res) {
						that.originalImage = res.tempFilePaths[0];
						that.generatedImage = null;
						that.qrcode = null;

						setTimeout(function() {
							// 获取图片宽高
							uni.getImageInfo({
								src: res.tempFilePaths[0],
								success: info => {
									// 原始图片的真实宽高
									that.imgWidth = info.width;
									that.imgheight = info.height;
									let ratio_w = that.originalWidth / info.width;
									let ratio_h = that.originalHeight / info.height;
									
									if (ratio_w > ratio_h) {
										let distance = (that.originalWidth - (that.originalHeight * info.width) / info.height) / 2;
										that.canvasTop = 0;
										that.canvasLeft = distance;
										that.canvasHeight = that.originalHeight;
										that.canvasWidth = (that.originalHeight / info.height) * info.width;
									} else {
										let distance = (that.originalHeight - (that.originalWidth / info.width) * info.height) / 2;
										that.canvasLeft = 0;
										that.canvasTop = distance;
										that.canvasWidth = that.originalWidth;
										that.canvasHeight = (info.width / info.width) * info.height;
									}
								},
								fail(e) {
									console.log(e);
								}
							})
						}, 100);
					}
				});
			},
			
			// 获取授权判断是否授权相册
			getSaveImg(){
				let that = this;
				uni.getSetting({
					success: (res)=>{
						if (!res.authSetting["scope.writePhotosAlbum"]){
							uni.showModal({
								title: '警告',
								content: '检测到您没您未授权相册权限,点击确定获取授权。',
								success: function (res) {
									if (res.confirm) {
										uni.authorize({
											scope: 'scope.writePhotosAlbum',
											success() {
												// 用户已经同意
												that.saveImgToAlbum(this.qrcode);
											},
											fail(err){
												console.log('用户拒绝授权相册') 
											}
										})		 
									}
								}
							})
						} else {
							that.saveImgToAlbum(this.qrcode);
						}
					 },
					 fail: () => {
						console.log('点击了拒绝');
					 }
				})
			},
			
			// 保存二维码图片到相册
			saveImgToAlbum() {
				var fileManager = uni.getFileSystemManager();
				fileManager.writeFile({
					filePath: wx.env.USER_DATA_PATH+'/img.jpg', // 指定图片的临时路径
					data: this.qrcode, // 要写入的文本或二进制数据
					encoding: 'base64', // 指定写入文件的字符编码
					success: res => {
						console.log('写入文件成功', res)
						console.log(wx.env.USER_DATA_PATH + '/img.jpg')
						uni.saveImageToPhotosAlbum({ // 保存图片到相册
							filePath: wx.env.USER_DATA_PATH + '/img.jpg',
							success: function (res) {
								console.log('保存成功', res)
								uni.showToast({
									title: '保存成功',
								})
							},
							fail: function (err) {
								console.log('保存失败', err)
							}
						})
					},
					file: err => {
						console.log('写入文件失败', err)
					}
				})
			},
			
			//把base64转换成图片
			getBase64ImageUrl(base64Url) {
				console.log('==========');
				console.log(base64Url);
				let that = this;
				//拿到后端给的base64字符串
				var shareQrImg = `data:image/jpg;base64,` + base64Url;
				base64src(shareQrImg, resCurrent => {
						that.generatedImage = resCurrent;
				});
			},
			
			// 去水印
			removeWaterMark() {
				let that = this;
				let FSM = wx.getFileSystemManager();
				FSM.readFile({
					filePath: that.originalImage,
					encoding: "base64",
					success: (res) => {
						that.handleRemoveWaterMark(res.data);
					},
					fail: res => {
						console.log("转换base失败");
					}
				})
			},
			
			// ---------------------调用网络接口-----------------------
			// 生成去水印的图片
			handleRemoveWaterMark(imageBase64) {
				let that = this;
				let access_token = uni.getStorageSync("access_token");
				wx.request({
					url: "https://aip.baidubce.com/rest/2.0/image-process/v1/inpainting?access_token="+access_token,
					data: JSON.stringify({
						// rectangle: [{width: 460, top: 744, height: 58, left: 0}],
						rectangle: [{width: 460, top: 740, height: 58, left: 0}],
						
						image: imageBase64
					}),
					header: { 'Content-Type': 'application/x-www-form-urlencoded' },
					method: 'POST',
					success(res) {
						console.log(res);
						// that.qrcode = res.data.image;
						that.getBase64ImageUrl(res.data.image);
					},
					fail(e) {
						console.log(e);
					}
				})
			}
		}
	}
</script>

<!-- // export default {
//   mounted() {
//     this.setPosition();
//   },
//   methods: {
//     setPosition() {
//       const image = uni.createSelectorQuery().select('.image');
//       const view = uni.createSelectorQuery().select('.view-element');
//       image.boundingClientRect(data1 => {
//         view.boundingClientRect(data2 => {
//           const top = data2.top - data1.top;
//           const left = data2.left - data1.left;
// 					console.log('top==', top);
// 					console.log('left==', left)
//           view.css({
//             top: top + 'px',
//             left: left + 'px'
//           });
//         }).exec();
//       }).exec();
//     }
//   }
// }
// </script> -->