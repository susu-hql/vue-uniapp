<template>
	<view class="media-box" style="width: 264px;height: 160px;" :start='startUrl' :change:start="xgplayerModule.startPlay">
		<view id="mse" :detail="videoData" :change:detail="xgplayerModule.initJs"></view>
	</view>
</template>

<script>
	// 逻辑层
	export default {
		props: ['id', 'videoData'],
		data(){
			return {
				startUrl: 'https://wethink-zs-180.cyberwisdom.net.cn/resource/resource/3052/1728525211024.mp4'
			}
		},
		mounted() {
		},
		methods: {
			onPlay(){
				console.log('响应视图层方法')
			}
		}
	}
</script>

<script module="xgplayerModule" lang="renderjs">
	// 视图层
	import FlvPlayer from 'xgplayer';
    export default{
		data(){
			return {
				xgPlayer: null
			}
		},
        mounted(){
			const _this = this
			console.log('我是逻辑层数据=1======', this.startUrl);
			console.log('我是父组件传过来的值========', this.videoData);
			
			_this.xgPlayer = new FlvPlayer({
				id: 'mse',
				url: this.videoData.videourl,
				width: 264,
  				height: 160
			});
			_this.xgPlayer.once('play',()=>{
				console.log('播放成功')
				// 调用逻辑层方法
				this.$ownerInstance.callMethod('onPlay')
			})
			_this.xgPlayer.on('error',(err)=>{
				console.log('播放出错', err)
				let videoErr = document.getElementById(`videoErr${detail.index}`)
				// 重新播放
				videoErr.onclick = function () {
					_this.xgPlayer.destroy()
					_this.initPlayer(detail,ownerInstance)
				}
			})
		},
		onunload() {
		},
        methods:{
			// 逻辑层触发视图层函数
			startPlay() {
				console.log('我是逻辑层数据；=1==', this.startUrl);
				// this.xgPlayer.play()
			}
        }
    }
</script>
