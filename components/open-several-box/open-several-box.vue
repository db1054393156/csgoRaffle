<template>
	<view class="openSeveralBox">
		<view class="openSeveralBox-main" :style="{height:height}">
			<view class="openSeveralBox-main-row" v-for="(item,index) in poolListAll" :key="index">
				<view class="openSeveralBox-main-row-list"
					:style="{transform: 'translateY('+translateY[index].translateY+'%)','transition-duration':translateY[index].duration+'ms'}">
					<view class="openSeveralBox-main-row-list-item" v-for="(v,k) in item" :key="index+'-'+k">
						<slot :item="v">
							<view>
								<image :src="v.img" style="width: 200rpx;height: 200rpx"></image>
								<view class="openSeveralBox-main-row-list-item-text line-1">
									{{v.name}}
								</view>
							</view>
						</slot>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			list: {
				type: Array,
				default () {
					return []
				}
			},
			height: {
				type: String,
				default: '256rpx'
			},
			jackpotLength: {
				type: Number,
				default: 50
			},
			rowLength: {
				type: Number,
				default: 2
			},
			delay: {
				type: Number,
				default: 500
			},
			// 总的摇奖时间 单位毫秒
			duration: {
				type: Number,
				default: 8000
			},
			startMusic:{
				type:String,
				default:''
			},
			endMusic:{
				type:String,
				default:''
			},
			isPlayAudio: {
				type: Boolean,
				default: true
			},
			// 抽奖结果
			result: {
				type: Array,
				default () {
					return []
				}
			},
			resultName: {
				type: String,
				default: 'id'
			},
		},
		data() {
			return {
				moveCss: '',
				poolListAll: [],
				translateY: [],
			}
		},
		watch: {
			list: {
				handler(val) {
					if (val.length > 0) {
						this.init()
					}
				},
				immediate: true
			}
		},
		methods: {
			init() {
				//设置中奖位置在
				this.luckyNums = this.getRand(this.jackpotLength - 15, this.jackpotLength - 10);
				const lotteryResult = this.generateLottery(this.jackpotLength);
				for (var i = 0; i < this.rowLength; i++) {
					this.translateY.push({
						translateY: 0,
						duration: 0
					})
					this.poolListAll.push(lotteryResult)
				}
			},
			start() {
				//设置中奖奖品
				if (this.rowLength == this.result.length) {
					let poolList = JSON.parse(JSON.stringify(this.poolListAll))
					let animatimonTime = this.duration - (this.delay * this.result.length)
					const newList = poolList.map((pool, index) => {
						let item = this.list.filter((item) => item[this.resultName] == this.result[index])[
						0]; //把奖励加到列表里
						pool.splice(this.luckyNums - 1, 0, item)
						setTimeout(() => {
							this.translateY.splice(index, 0, {
								translateY: (this.luckyNums - 1) * -100,
								duration: animatimonTime
							})
						}, index * this.delay)
						return pool
					})
					this.poolListAll = newList
					this.playVoice1()
					setTimeout(()=>{
						this.$emit('finsh')
					},this.duration+100)
				} else {
					uni.showToast({
						title: '中奖结果与奖池个数不一致'
					})
				}
			},
			//指定范围内容获取随机数
			getRand(start, end) {
				return Math.floor(Math.random() * (end - start + 1) + start);
			},
			// 生成符合概率的滚动抽奖数据的函数
			generateLottery(num) {
				const result = [];
				const probabilityData = this.generateProbabilityData();
				for (let i = 0; i < num; i++) {
					const randomIndex = Math.floor(Math.random() * probabilityData.length);
					const randomData = probabilityData[randomIndex];
					result.push(randomData);
				}
				return result;
			},
			// 根据概率生成抽奖数据的函数
			generateProbabilityData() {
				const result = [];
				let totalProbability = 0;
				this.list.forEach(data => {
					if (data[this.probability]) {
						totalProbability += data[this.probability] * 1
					} else {
						totalProbability += 1
					}
				});
				for (let i = 0; i < this.list.length; i++) {
					const data = this.list[i];
					let pro = data[this.probability] ? data[this.probability] : 1
					const count = Math.round(pro * 1 / totalProbability * 60);
					for (let j = 0; j < count; j++) {
						result.push(data);
					}
				}
				return result;
			},
			playVoice1() {
				if (!this.isPlayAudio) {
					return
				}
				if(this.startMusic==''){
					return
				}
				// #ifdef H5
				const sound = new Howl({
				  src: [this.startMusic],
				  autoplay: true,
				});
				sound.play();
				sound.on('end', ()=>{
				  this.playVoice2()
				});
				// #endif
				// #ifndef H5
				var innerAudioContext = uni.createInnerAudioContext();
				innerAudioContext.autoplay = true
				innerAudioContext.src = this.startMusic;
				innerAudioContext.onPlay(() => {
					console.log('开始播放');
				});
				innerAudioContext.onError((res) => {
					console.log(res.errMsg);
					console.log(res.errCode);
					innerAudioContext.destroy()
				});
				innerAudioContext.onEnded(()=>{
					this.playVoice2()
					innerAudioContext.destroy()
				})
				innerAudioContext.play()
				// #endif
				
				
			},
			playVoice2() {
				// #ifdef H5
				const sound = new Howl({
				  src: [this.endMusic],
				  autoplay: true,
				});
				sound.play();
				// #endif
				// #ifndef H5
				var innerAudioContext = uni.createInnerAudioContext();
				innerAudioContext.autoplay = true
				innerAudioContext.src = this.endMusic;
				innerAudioContext.onPlay(() => {
					console.log('开始播放');
				});
				innerAudioContext.onError((res) => {
					console.log(res.errMsg);
					console.log(res.errCode);
					innerAudioContext.destroy()
				});
				innerAudioContext.onEnded(()=>{
					innerAudioContext.destroy()
				})
				innerAudioContext.play()
				// #endif
			},
		}
	}
</script>

<style lang="scss" scoped>
	.line-1 {
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
	}

	.openSeveralBox {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;

		.openSeveralBox-main {
			display: flex;
			flex-direction: row;
			align-items: center;

			.openSeveralBox-main-row {
				display: flex;
				flex-direction: column;
				height: 100%;
				overflow: hidden;
				margin: 0 4rpx;

				.openSeveralBox-main-row-list {
					flex: 1;
					display: flex;
					flex-direction: column;
					height: 100%;
					transition-property: transform;
					transition-timing-function: cubic-bezier(0.2, 0, 0.1, 1);

					.openSeveralBox-main-row-list-item {
						flex-shrink: 0;
						display: flex;
						flex-direction: column;
						background: #222225;
						border-radius: 10rpx;
						align-items: center;
						color: #fff;
						min-height: 100%;
						flex: 1;

						.openSeveralBox-main-row-list-item-text {
							flex: 1;
							width: 200rpx;
							color: #fff;
							padding: 0 10rpx;
						}
					}
				}
			}
		}
	}
</style>
