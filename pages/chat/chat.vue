<template>
	<view>对话</view>
</template>

<script>
	export default {
		data() {
			return {
				stv: {
					offsetX: 0,
					offsetY: 0,
					zoom: false, //是否缩放状态
					distance: 0,  //两指距离
					scale: 1,  //缩放倍数
				}
			}
		},
		
		methods: {
			// 触摸开始
			touchstartCallback: function(e) {
			  console.log('touchstartCallback');
			  console.log(e);
				let that = this;
			  if (e.touches.length === 1) {
			    let {clientX, clientY} = e.touches[0];
			    this.startX = clientX;
			    this.startY = clientY;
			    this.touchStartEvent = e.touches;
			  } else {
			    let xMove = e.touches[1].clientX - e.touches[0].clientX;
			    let yMove = e.touches[1].clientY - e.touches[0].clientY;
			    let distance = Math.sqrt(xMove * xMove + yMove * yMove);
					this.stv.distance = distance;
					this.stv.zoom = true; //缩放状态
			  }
			},
			// 触摸移动中
			touchmoveCallback: function(e) {
			  //console.log('touchmoveCallback');
			  //console.log(e);
			
			  if (e.touches.length === 1) {
			    // 单指移动
			    if (this.stv.zoom) {
			      // 缩放状态，不处理单指
			      return ;
			    }
			    let {clientX, clientY} = e.touches[0];
			    let offsetX = clientX - this.startX;
			    let offsetY = clientY- this.startY;
			    this.startX = clientX;
			    this.startY = clientY;
			    this.stv.offsetX += offsetX;
			    this.stv.offsetY += offsetY;
			    this.stv.offsetLeftX = -this.stv.offsetX;
			    this.stv.offsetLeftY = -this.stv.offsetLeftY;
			  } else {
			    // 双指缩放
			    let xMove = e.touches[1].clientX - e.touches[0].clientX;
			    let yMove = e.touches[1].clientY - e.touches[0].clientY;
			    let distance = Math.sqrt(xMove * xMove + yMove * yMove);
			
			    let distanceDiff = distance - this.stv.distance;
			    let newScale = this.stv.scale + 0.005 * distanceDiff;
			
					this.stv.distance = distance;
					this.stv.scale = newScale;
			  }
			},
			// 触摸结束
			touchendCallback: function(e) {
			  console.log('touchendCallback');
			  console.log(e);
			
			  if (e.touches.length === 0) {
			    this.stv.zoom = false; //重置缩放状态
			  }
			},
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
