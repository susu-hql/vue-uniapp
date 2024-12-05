<template>
	<view class="audio-content">
		<!-- #ifdef H5 -->
			<video
				v-if="fileUrl"
				id="le-audio" 
				style="display: none;"
				:autoplay="autoplay"
				:src="fileUrl" 
				@error="onError" 
				@timeupdate="setPlayData"
				@loadedmetadata="onCanplay"
				@play="onPlay"
				@pause="onPause"
				@ended="onEnded"
			>
			</video>
		<!-- #endif -->
		<view class="audio-wrapper">
			<view class="audio-left">
				<view class="play-box">
					<!-- :style="{backgroundColor:isPaused?'#00CA86':'#BDBDBD'}"  -->
					<view class="play-btn" style="background: none;" @click="onPlayPause">
						<image v-show="isPaused" src="./icon/suspend.svg"></image>
						<image v-show="!isPaused" src="./icon/start.svg"></image>
					</view>
				</view>
			</view>
			<view class="audio-right">
				<!-- 调速拉条区 -->
				<view class="audio-progress">
					<!-- 显示时间进度区 -->
					<text style="color: #888;">{{minTimeFmt(currentTime)}}</text>
					<view class="audio-bar">
						<slider 
							class="audio-slider" 
							:value="currentTime" 
							:min="0" 
							:max="duration" 
							@change="sliderChange"
							@changing="sliderChanging" 
							activeColor="#075ebb" 
							backgroundColor="#DAE0E6" 
							block-color="#075ebb"
							block-size="12" />
					</view>
					<text style="color: #888;">{{minTimeFmt(duration)}}</text>
				</view>
				<view class="audio-controller-box">
					<!-- <image src="./icon/get-back.svg" @click="onSeek(-15)"></image>
					<image src="./icon/fast-forward.svg" @click="onSeek(15)"></image> -->
					<view class="speed-box">
						<text v-if="showAudioSpeedIcon" class="speed-text" @tap="onSpeed(0)">x{{speed}}</text>
					</view>
				</view>
				<view v-if="speedModal" class="speed-modal">
					<view @tap="onSpeed(item)" :style="speed===item?'color: #075ebb':''" v-for="item in speedList" class="text">{{item}}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	let innerAudioContext = null;
	export default {
		name: "le-audio",
		props: {
			/**
			 * audioUrl 播放路径
			 * */
			 audioUrl: {
				type: String,
				default: ''
			},
			speedList: {
				type: Array,
				default: () => {
					return [0.5, 1, 1.5, 2]
				}
			},
			// 当前播放的位置索引
			activeIndex: {
				type: Number,
				default: -1
			},
			// 是否显示播放倍速
			showAudioSpeedIcon: {
				type: Boolean,
				default: false
			},
			// 是否自动播放
			autoplay:{
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				isPaused: true, //是否暂停中
				duration: 0, //音频时长
				currentTime: 0, //当前时长
				speed: 1, //倍速
				isSlidering: false, //是否移动中
				isEndAcudio: false, //最后一个音频结束
				fileUrl: '',
				isAutoplay:false,
				speedModal:false
			}
		},
		watch: {
			audioUrl: {
				handler: function() {
					this.fileUrl = this.audioUrl;
					this.setAudioInfo();
				},
				deep: true
			}
		},
		computed: {
		},
		mounted() {
			this.$nextTick(() => {
				this.startPlay();
				this.fileUrl = this.audioUrl;
				this.setAudioInfo();
			})
		},
		beforeDestroy() {
		},
		methods: {
			minTimeFmt(val) {
				let minute = parseInt(val / 60);
				let seconds = Math.ceil(val % 60)
				return `${minute>=10?minute:'0'+minute}:${seconds>=10?seconds:'0'+seconds}`
			},
			beforeLeave() {
				// #ifdef H5
				innerAudioContext.pause();
				// #endif
				// #ifndef H5
				innerAudioContext.pause();
				// innerAudioContext.destroy();
				// #endif
			},
			startPlay() {
				// #ifdef H5
					innerAudioContext = uni.createVideoContext('le-audio');
				// #endif
				// #ifdef MP-ALIPAY || MP-LARK
					innerAudioContext = uni.createInnerAudioContext();
				// #endif
				// #ifndef H5 || MP-ALIPAY || MP-LARK
					innerAudioContext = uni.getBackgroundAudioManager();
				// #endif
				/*
				play		播放（H5端部分浏览器需在用户交互时进行）	
				pause		暂停	
				stop		停止	
				seek	position	跳转到指定位置，单位 s	
				destroy		销毁当前实例
				*/
				innerAudioContext.startTime = 0; //开始播放的位置（单位：s）
				console.log('innerAudioContext=======', innerAudioContext);
				this.setAudioInfo();
				// #ifdef H5
					innerAudioContext.playbackRate(1.0);
				// #endif
				// #ifndef H5
					innerAudioContext.playbackRate = this.speed; //放的倍率。可取值：0.5/0.8/1.0/1.25/1.5/2.0
					// 音频播放事件
					innerAudioContext.onPlay(() => {
						this.onPlay();
					});
					// 音频暂停事件
					innerAudioContext.onPause(() => {
						this.onPause();
					});
					// 音频进入可以播放状态，但不保证后面可以流畅播放
					innerAudioContext.onCanplay(() => {
						this.onCanplay();
					});
					// 音频自然播放结束事件
					innerAudioContext.onEnded(() => {
						this.onEnded();
					});
					// 音频播放错误事件
					innerAudioContext.onError((res) => {
						this.onError(res)
					});
					// 音频播放进度更新事件
					innerAudioContext.onTimeUpdate(() => {
						this.setPlayData();
					});
				// #endif
			},
			// 暂停播放切换
			onPlayPause() {
				if (this.isPaused) {
					if (this.isEndAcudio) {
						innerAudioContext.src = encodeURI(this.fileUrl).replace(/\+/g, "%2B"); //音频的数据链接，用于直接播放。
					}
					innerAudioContext.play()
					this.isPaused = false;
				} else {
					innerAudioContext.pause()
					this.isPaused = true;
				}
			},
			// 调整播放倍速
			onSpeed(val) {
				if (val) {
					this.speed = val;
					// #ifdef H5
					innerAudioContext.playbackRate(val);
					// #endif
					// #ifndef H5
						innerAudioContext.playbackRate = val;
					// #endif
					this.speed = val;
					this.speedModal = false;
				} else {
					this.speedModal = !this.speedModal;
				}
			},
			// 调整播放位置
			onSeek(num) {
				if (!innerAudioContext.currentTime) return;
				let seekNum = num + innerAudioContext.currentTime;
				this.isSlidering = true;
				if (seekNum <= 0) {
					// 调整后时间小于0
					this.currentTime = 0;
					innerAudioContext.seek(0)
				} else if (seekNum > innerAudioContext.duration) {
					// 调整后时间大于总时长
					this.currentTime = innerAudioContext.duration;
					innerAudioContext.seek(innerAudioContext.duration)
				} else {
					this.currentTime = seekNum;
					innerAudioContext.seek(seekNum)
				}
			},
			// 滑块滚动到的位置
			sliderChange(e) {
				this.isSlidering = false;
				this.currentTime = e.detail.value;
				innerAudioContext.seek(e.detail.value);
			},
			// 滑块滚动到的位置 实时
			sliderChanging(e) {
				this.isSlidering = true;
				this.currentTime = e.detail.value;
			},
			// 设置以及转换信息
			setPlayData(event) {
				// #ifndef H5
					if (this.isSlidering) return;
					if (!innerAudioContext.duration && !innerAudioContext.currentTime) return;
					this.duration = innerAudioContext.duration || 0;
					this.currentTime = innerAudioContext.currentTime || 0;
				// #endif
				// #ifdef H5
					if (this.isSlidering) return;
					this.duration = event.detail.duration  || 0;
					this.currentTime = event.detail.currentTime || 0;
				// #endif
			},
			// 设置播放
			setAudioInfo() {
				if (innerAudioContext && this.fileUrl) {
					innerAudioContext.src = encodeURI(this.fileUrl).replace(/\+/g, "%2B"); //音频的数据链接，用于直接播放。
					if(this.isAutoplay||this.autoplay){
						innerAudioContext.play();
					}else{
						setTimeout(()=>{
							innerAudioContext.pause();
						},100)
						this.isAutoplay = true;
					}
				}
				if (!this.fileUrl) {
					this.duration = 0;
					this.currentTime = 0;
					this.isPaused = true;
					innerAudioContext.pause();
				}
			},
			// 播放事件
			onPlay(){
				// console.log('音频播放事件');
				this.isEndAcudio = false;
				this.isSlidering = false;
				this.isPaused = false;
			},
			// 暂停事件
			onPause(){
				// console.log("音频暂停事件");
				this.isPaused = true;
			},
			// 播放结束事件
			onEnded(){
				console.log("音频自然播放结束事件");
			},
			// 音频进入可以播放状态
			onCanplay(event){
				// console.log('音频进入可以播放状态，但不保证后面可以流畅播放');
				// console.log("音频时长：", innerAudioContext.duration)
				this.isSlidering = false;
				this.setPlayData(event);
			},
			// 播放失败
			onError(res) {
				// console.log("音频播放错误事件", res);
				this.duration = 0;
				this.currentTime = 0;
				this.isPaused = true;
				innerAudioContext.pause();
			},
			
		}
	}
</script>

<style lang="scss" scoped>
	.audio-content {
		// width: 100vw;
		// padding: 0 64rpx;
		box-sizing: border-box;
		display: flex;
    	justify-content: center;
		
		.audio-wrapper{
			max-width: 100%;
			min-width: 95%;
			height: 130rpx;
			background: #fcfcfc;
			border: 2rpx solid #e0e0e0;
			border-radius: 5rpx;
			display: inline-block;
			// overflow: hidden;
			.audio-left{
			    width: 130rpx;
				height: 130rpx;
				float: left;
				// background-color: rgba(230,230,230,0.7);
				background-color: #666;
				background-size: 100% 100%;
				background-position: 50% 50%;

				.play-box {
					display: flex;
					align-items: center;
					width: 100%;
					height: 100%;
					justify-content: center;
					.play-btn {
						width: 96rpx;
						height: 96rpx;
						display: flex;
						align-items: center;
						justify-content: center;
						background: #475266;
						border-radius: 50%;
						image {
							width: 96rpx;
							height: 96rpx;
						}
					}
				}
			}
			.audio-right{
				box-sizing: border-box;
				height: 130rpx;
				margin-left: 130rpx;
				padding: 22rpx 33rpx 27rpx 30rpx;
				// overflow: hidden;
				position: relative;

				.audio-progress {
					display: flex;
					align-items: center;
					justify-content: space-between;

					image {
						width: 48rpx;
						height: 48rpx;
					}

					.audio-bar {
						flex: 1;
						padding: 0 26rpx;
						box-sizing: border-box;

						.audio-slider {
							margin: 0 !important;
						}
					}
				}
				.audio-controller-box{
					display: flex;
					align-items: center;
					justify-content: flex-end;
					// justify-content: space-around;
					margin-top: 5px;
					image {
						width: 48rpx;
						height: 48rpx;
					}
					.speed-box{
						position: relative;
						width: 48rpx;
						height: 48rpx;
						margin-left: 20rpx;
						line-height: 48rpx;
						
						image {
							width: 48rpx;
							height: 48rpx;
						}
						.speed-text {
							position: absolute;
							top: 0;
							right: 0;
							color: #475266;
							font-size: 28rpx;
							font-weight: 600;
						}
					}
				}
				.speed-modal{
					position: absolute;
					background: white;
					border: 1px solid #eee;
					width: 100px;
					right: 0;
					text-align: center;
					.text{
					    padding: 10rpx 0;
					}
				}
			}
		}
	}
</style>