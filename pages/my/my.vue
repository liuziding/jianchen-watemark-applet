<template>
	<view class="my">
		<view class="my-head">
			<view class="my_avatar_icon"></view>
			<view class="my_avatar_text">登录</view>
		</view>
		<!-- <MyList /> -->
	</view>
</template>

<script>
	// import MyList from '@/components/my-list/my-list.vue';
	export default {
		data() {
			return {
				
			}
		},
		components: {
			// MyList,
		},
		methods: {
			
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #F5F5F5;
	}
	.my {
		.my-head {
			width: 100%;
			height: 300rpx;
			background-color: #408CE7;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			.my_avatar_icon {
				background-color: aquamarine;
				border-radius: 50%;
				height: 100rpx;
				width: 100rpx;
			}
			.my_avatar_text {
				color: #FFF;
				font-size: 28rpx;
				margin-top: 6rpx;
			}
		}
	}
</style>
