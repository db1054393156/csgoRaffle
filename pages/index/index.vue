<template>
	<view class="content">
		<view class="">
			<open-one-box ref="oneBox" :list="box_skins" result="597" startMusic="static/audio/open8.mp3" endMusic="static/audio/14131.mp3" @finsh="finsh">
				<!-- 可以使用插槽自定内容 -->
				<!-- <template v-slot="{ item }">
					<image :src="item.img" style="width: 100rpx;height: 100rpx;"></image>
				</template> -->
			</open-one-box>
			<view style="display: flex;align-items: center;justify-content: center;margin-top: 30rpx;" >
				<button @click="startOne" style="width: 200rpx;background: #278891;">开始</button>
			</view>
		</view>
		
		<view style="margin-top: 80rpx;">
			<open-several-box ref="severalBox" :list="box_skins" :result="[597,509]" startMusic="static/audio/open8.mp3" endMusic="static/audio/14131.mp3" @finsh="finsh">
				<!-- 可以使用插槽自定内容 -->
				<!-- <template v-slot="{ item }">
					<image :src="item.img" style="width: 200rpx;height: 200rpx;"></image>
				</template> -->
			</open-several-box>
			<view style="display: flex;align-items: center;justify-content: center;margin-top: 30rpx;" >
				<button @click="startSeveral" style="width: 200rpx;background: #278891;">开始</button>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				box_skins: [{
						"id": 1147,
						"name": "M4A4 | 主机 (崭新出厂)",
						"img": require('static/test1.png'),
						"probability": 10,
					},
					{
						"id": 597,
						"name": "印花 | Natus Vincere | 2020 RMR",
						"img": require('static/test2.png'),
						"probability": 0,
					},
					{
						"id": 509,
						"name": "M4A4 | 变频器 (崭新出厂)",
						"img": require('static/test3.png'),
						"probability": 12,
					},
					{
						"id": 1234,
						"name": "M4A4 | 都市 DDPAT (久经沙场)",
						"img": require('static/test4.png'),
						"probability": 12,
					}
				]
			}
		},
		onLoad() {
		},
		methods: {
			startOne(){
				this.$refs.oneBox.start()
			},
			startSeveral(){
				this.$refs.severalBox.start()
			},
			finsh(){
				uni.showToast({
					title:"抽奖完成"
				})
			}
		}
	}
</script>

<style>
</style>
