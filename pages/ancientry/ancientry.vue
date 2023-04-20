<template>
	<view class="ancientry">
		1234
		<!-- <view class="ancientry_original">
			<template v-if="originalImage">
				<image :src="originalImage" mode="aspectFit" />
				<view class="canvas_main" :style="{position: 'absolute', left:canvasLeft+'px', top:canvasTop+'px', right:canvasRight+'px', bottom:canvasBottom+'px'}">
					1341234213
				</view>
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
		</view> -->
	</view>
</template>

<script>
	import base64src from '@/utils/base64.js';
	export default {
		data() {
			return {
				imgWidth: 0, 								// 原始图片宽度
				imgHeight: 0, 							// 原始图片高度
				canvasLeft: 0, 						// 画布距离左边大小
				canvasTop: 0, 							// 画布距离顶部大小
				canvasRight: 0, 						// 画布距离右边大小
				canvasBottom: 0, 					// 画布距离底部大小
				originalWidth: 0, 					// 原始盒子宽度
				originalHeight: 0,					// 原始盒子高度
				originalImage: null,
				generatedImage: null,
				qrcode: null,
			}
		},
		onLoad() {
			// this.getBase64ImageUrl(this.qrcode);
			this.handleGetAccessToken();
			
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
										let distance = (that.originalWidth - (that.originalHeight * that.imgWidth) / info.height) / 2;
										that.canvasTop = 0;
										that.canvasBottom = 0;
										that.canvasLeft = distance;
										that.canvasRight = distance;
									} else {
										let distance = (that.originalHeight - (that.originalWidth / info.width) * info.height) / 2;
										that.canvasLeft = 0;
										that.canvasRight = 0;
										that.canvasTop = distance;
										that.canvasBottom = distance;
									}
								},
								fail(e) {
									console.log(e);
									debugger;
								}
							})
						}, 100);
					}
				});
			},
			
			// 保存图片
			// savePhoto() {
			// 	uni.saveVideoToPhotosAlbum({
			// 		filePath: this.getBase64ImageUrl,
			// 		success() {
			// 			uni.showToast({
			// 				title: "保存成功"
			// 			});
			// 		},
			// 		fail() {
			// 			uni.showToast({
			// 				title: "保存失败",
			// 				icon: "none"
			// 			})
			// 		}
			// 	})
			// },
			
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
			// 调用百度图像增强与特效接口，获取access_token
			handleGetAccessToken() {
				uni.request({
					url: "https://aip.baidubce.com/oauth/2.0/token",
					data: {
						grant_type: "client_credentials",
						client_id: "c5V1SCzAG45uItEaLmItKT3L",
						client_secret: "MMYwGmRaRG4P6j7wkz0j89y2aTKjtec0"
					},
					success: (res) => {
						console.log(res);
						console.log(res.data.access_token);
						const { access_token } = res.data;
						if (access_token) {
							uni.setStorageSync("access_token", access_token);
							// uni.setStorage({
							// 	key: "access_token",
							// 	data: access_token
							// });
						} else {
							uni.setStorageSync("access_token", null);
							// uni.setStorage({
							// 	key: "access_token",
							// 	data: null
							// });
						}
					},
					fail() {
						console.log("获取access_token失败！");
					}
				});
			},
			
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
				});
			}
		}
	}
</script>

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
			position: relative;
			border: 1px solid green;
			.canvas_main {
				// position: absolute;
				// left: 40px;
				// top: 20px;
				// bottom: 20px;
				// right: 20px;
				border: 1px solid red;
			}
			.angle {
				border-right: 8rpx solid #999;
				border-top: 8rpx solid #999;
				width: 60rpx;
				height: 60rpx;
			}
			.left-top-angle {
				position: absolute;
				left: 20rpx;
				top: 20rpx;
				transform: rotate(270deg);
			}
			.left-bottom-angle {
				position: absolute;
				left: 20rpx;
				bottom: 20rpx;
				transform: rotate(180deg);
			}
			.right-top-angle {
				position: absolute;
				right: 20rpx;
				top: 20rpx;
			}
			.right-bottom-angle {
				position: absolute;
				right: 20rpx;
				bottom: 20rpx;
				transform: rotate(90deg);
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
				// &:toach {
				// 	border: 1px solid #476DBE;
				// 	background-color: #476DBE;
				// }
			}
		}
	}
</style>
