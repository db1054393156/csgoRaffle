<template>
	<view class="openOneBox">
		<view class="openOneBox-animation" >
			<view style="flex: 1;display: flex;flex-direction: row;" :style="moveCss">
				<view class="openOneBox-animation-item" :class="{'resultItem':index==luckyNums-1}"  v-for="(item,index) in poolList" :key="index">
					<slot :item="item">
						<view class="openOneBox-animation-item-main">
							<image :src="item.img" style="width: 200rpx;height: 200rpx"></image>
							<view class="openOneBox-animation-item-main-text line-1">
								{{item.name}}
							</view>
						</view>
					</slot>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	// #ifdef H5
	import {
		Howl
	} from 'howler'
	// #endif
	
	/**
	 * 一个盒子的开奖
	 * @property {Array} list 参与抽奖的奖品
	 * @property {Number} jackpotLength 抽奖的奖池长度，为了实现开奖动画
	 * @property {String} probability 奖品的概率字段
	 */
	export default {
		props:{
			list:{
				type:Array,
				default(){
					return []
				}
			},
			jackpotLength:{
				type:Number,
				default:90
			},
			probability:{
				type:String,
				default:'probability'
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
				type: [Number,String],
				default: null
			},
			resultName:{
				type:String,
				default:'id'
			},
			
		},
		data() {
			return {
				moveCss:'',//开始动画是的样式
				luckyNums: 0, //中奖位置
				poolList:[]
			}
		},
		watch:{
			list:{
				handler(val){
					if(val.length>0){
						this.init()
					}
				},
				immediate:true
			}
		},
		methods: {
			init(){
				//设置中奖位置在
				this.luckyNums = this.getRand(this.jackpotLength-15, this.jackpotLength-10);
				const lotteryResult = this.generateLottery(this.jackpotLength);
				this.poolList = lotteryResult
			},
			start(){
				//设置中奖奖品
				let item = this.list.filter((item) => item[this.resultName] == this.result)[0]; //把奖励加到列表里
				this.poolList.splice(this.luckyNums - 1, 0, item)
				//开始动画
				this.$nextTick(()=>{
					const query = uni.createSelectorQuery().in(this);
					query.selectAll('.openOneBox-animation,.resultItem').boundingClientRect(data => {
						const sum = data.reduce((accumulator, currentValue) => {
						    const { left, width } = currentValue;
						    return {
						    left: Math.abs(accumulator.left - left),
						    width: Math.abs(accumulator.width - width)
						};
						}, { left: 0, width: 0 });
						let scrollCenter=sum.left-sum.width/2
						this.playVoice1()
						this.moveCss =
							`margin-left:${-scrollCenter}px;transition:all ${this.duration}ms cubic-bezier(.1,.59,.1,.9)`;
						setTimeout(()=>{
							this.$emit('finsh')
						},this.duration+100)
					}).exec();
				})
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
					if(data[this.probability]){
						totalProbability += data[this.probability] * 1
					}else{
						totalProbability += 1
					}
				});
				for (let i = 0; i < this.list.length; i++) {
					const data = this.list[i];
					let pro=data[this.probability]?data[this.probability]:1
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
	.line-1{
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
	}
	.openOneBox {
		display: flex;
		flex-direction: column;

		.openOneBox-animation {
			display: flex;
			flex-direction: row;
			align-items: center;
			flex-wrap: nowrap;
			overflow: hidden;

			.openOneBox-animation-item {
				display: flex;
				flex-direction: column;
				min-width: 0;
				.openOneBox-animation-item-main{
					display: flex;
					flex-direction: column;
					min-width: 0;
					margin-right: 8rpx;
					background: #222225;
					border-radius: 10rpx;
					padding: 10rpx;
					.openOneBox-animation-item-main-text{
						flex: 1;
						width: 200rpx;
						color: #fff;
					}
				}
			}
		}
	}
</style>
