<template>
	<view>
		<movable-area>
			<movable-view
				:x="x" :y="y"
				:scale="true"
				:scale-min="scaleMin"
				:scale-max="scaleMax"
				direction="all">text</movable-view>
		</movable-area>
	</view>
</template>

<script>
	export default {
		 // :style="{height: boxHeight+'px', width: boxWidth+'px'}"
		data() {
			return {
				x: 10,
				y:10,
				boxWidth: 100,
				boxHeight: 100,
				scaleMin: 0.5,
				scaleMax: 5
			}
		},
		
		methods: {
		}
	}
</script>

<style lang="scss" scoped>
	movable-area {
	  height: 400rpx;
	  width: 400rpx;
	  margin: 50rpx;
	  background-color: #ccc;
	  overflow: hidden;
	}
	
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
</style>
