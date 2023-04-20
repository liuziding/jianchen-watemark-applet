<template>
  <div class="canvas-container">
		<!-- <canvas class="canvas" id="myCanvas" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd" ref="canvas"></canvas> -->
		<canvas class="canvas" type="2d" id="canvas" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd"></canvas>
    <div class="button-container">
      <button @click="uploadImage">上传图片</button>
      <button @click="addRect">添加矩形框</button>
      <button @click="confirmPosition">确定位置</button>
    </div>
  </div>
</template>

<script>
	export default {
		data() {
			return {
				canvas: {},
				ctx: null,
				startX: 0,
				startY: 0,
				endX: 0,
				endY: 0,
				imgWidth: 0,
				imgHeight: 0,
				rectX: 0,
				rectY: 0,
				rectWidth: 0,
				rectHeight: 0,
				rectMinSize: 10,
				rectMaxX: 0,
				rectMaxY: 0,
				rectMinX: 0,
				rectMinY: 0,
				rectMoved: false,
				rectSelected: false
			}
		},
		mounted() {
			// 获取canvas对象和画布上下文
			// this.canvas = uni.createSelectorQuery().select('#myCanvas')
			// this.ctx = this.canvas.getContext('2d')
			
			uni.createSelectorQuery()
					.select("#canvas")
					.fields({ node: true, size: true })
					.exec(async (res) => {
						debugger;
						const cvs = res[0].node;
						this.ctx = cvs.getContext('2d');
					});
		},
		methods: {
			touchStart(event) {
				// 处理用户点击操作
				this.startX = event.touches[0].clientX
				this.startY = event.touches[0].clientY
				this.rectSelected = this.isRectSelected(this.startX, this.startY)
			},
			touchMove(event) {
				// 处理用户拖动和缩放矩形框的操作
				this.endX = event.touches[0].clientX
				this.endY = event.touches[0].clientY
				if (this.rectSelected) {
					this.moveRect(this.endX - this.startX, this.endY - this.startY)
				} else {
					this.rectWidth = this.endX - this.startX
					this.rectHeight = this.endY - this.startY
				}
				this.startX = this.endX
				this.startY = this.endY
				this.drawRect()
			},
			touchEnd(event) {
				// 处理用户结束操作
				this.rectMoved = false
			},
			// 上传图片
			uploadImage() {
				let that = this;
				uni.chooseImage({
					success: res => {
						let tempFilePath = res.tempFilePaths[0]
						// 获取图片宽高
						uni.getImageInfo({
							src: tempFilePath,
							success: info => {
								// 设置canvas画布宽高
								that.canvas.width = info.width
								that.canvas.height = info.height
								that.imgWidth = info.width
								that.imgHeight = info.height
								console.log(that.ctx); 
								
								// 绘制图片
								that.ctx.drawImage(tempFilePath, 0, 0, info.width, info.height)
							}
						})
					}
				});
			},
			uploadImage2() {
				uni.chooseImage({
					success: res => {
						let tempFilePath = res.tempFilePaths[0]
						// 获取图片宽高
						uni.getImageInfo({
							src: tempFilePath,
							success: info => {
								this.imgWidth = info.width
								this.imgHeight = info.height
								// 设置canvas画布宽高
								this.canvas.width = this.imgWidth
								this.canvas.height = this.imgHeight
								// 绘制图片
								this.ctx.drawImage(tempFilePath, 0, 0, this.imgWidth, this.imgHeight)
							}
						})
					}
				})
			},
			addRect() {
				this.resetRect()
				this.drawRect()
			},
			resetRect() {
				// 初始化矩形框位置和大小
				this.rectX = this.imgWidth / 2
				this.rectY = this.imgHeight / 2
				this.rectWidth = this.rectMinSize
				this.rectHeight = this.rectMinSize
				this.rectMaxX = this.imgWidth - this.rect
					this.rectMaxY = this.imgHeight - this.rectMinSize
					this.rectMinX = this.rectMinSize
					this.rectMinY = this.rectMinSize
				},
			drawRect() {
			  // 绘制矩形框和边框
			  this.ctx.clearRect(0, 0, this.imgWidth, this.imgHeight)
			  this.ctx.drawImage(this.canvas, 0, 0, this.imgWidth, this.imgHeight)
			  this.ctx.fillStyle = 'rgba(255, 0, 0, 0.2)'
			  this.ctx.fillRect(this.rectX - this.rectWidth / 2, this.rectY - this.rectHeight / 2, this.rectWidth, this.rectHeight)
			  this.ctx.strokeStyle = 'red'
			  this.ctx.lineWidth = 2
			  this.ctx.strokeRect(this.rectX - this.rectWidth / 2, this.rectY - this.rectHeight / 2, this.rectWidth, this.rectHeight)
			},
			isRectSelected(x, y) {
			  // 判断是否选中矩形框
			  let left = this.rectX - this.rectWidth / 2
			  let right = this.rectX + this.rectWidth / 2
			  let top = this.rectY - this.rectHeight / 2
			  let bottom = this.rectY + this.rectHeight / 2
			  if (x >= left && x <= right && y >= top && y <= bottom) {
			    return true
			  } else {
			    return false
			  }
			},
			moveRect(dx, dy) {
			  // 移动矩形框
			  let newX = this.rectX + dx
			  let newY = this.rectY + dy
			  if (newX >= this.rectMinX && newX <= this.rectMaxX && newY >= this.rectMinY && newY <= this.rectMaxY) {
			    this.rectX = newX
			    this.rectY = newY
			    this.rectMoved = true
			  }
			},
			confirmPosition() {	
			  // 确定位置并输出结果
			  if (this.rectMoved) {
			    let left = this.rectX - this.rectWidth / 2
			    let top = this.rectY - this.rectHeight / 2
			    console.log('矩形框左上角坐标：(' + left + ', ' + top + ')')
			    console.log('矩形框宽高：(' + this.rectWidth + ', ' + this.rectHeight + ')')
			  }
			}
		}
	}
</script>
	
<style lang="scss" scoped>
	.canvas-container {
	  position: relative;
	  width: 100%;
	  height: 100%;
	  display: flex;
	  flex-direction: column;
	  justify-content: center;
	  align-items: center;
	}
	
	.canvas {
	  max-width: 100%;
	  max-height: 80vh;
	  border: 1px solid #ccc;
	}
	
	.button-container {
	  margin-top: 20px;
	  display: flex;
	  flex-direction: row;
	  justify-content: space-evenly;
	  width: 100%;
	}
	
	button {
	  padding: 10px;
	  border-radius: 5px;
	  border: none;
	  color: #fff;
	  background-color: #333;
	  cursor: pointer;
	  transition: all 0.2s ease-in-out;
	}
	
	button:hover {
	  background-color: #555;
	}
	
	button:active {
	  transform: translateY(2px);
	}
</style>
