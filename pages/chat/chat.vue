<template>
	<view>
		<movable-area>
			<movable-view
				:x="x"
				:y="y"
				scale="true"
				damping="900"
				friction="100"
				bindscale="myscaleevent"
				bindchange="mychangeevent"
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
			}
		},
		
		methods: {
			mychangeevent(e) {
				debugger;
				console.log("我被移动了", e);
			},
			myscaleevent(e) {
				debugger;
				console.log("我被移动了12", e);
			}
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
	  height: 180rpx;
	  width: 180rpx;
	  // background: #1AAD19;
		border: 1px solid red;
	  color: #fff;
	}
</style>
