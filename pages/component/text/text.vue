<!-- monitor.vue -->
<template>
	<view class="uni-padding-wrap monitor_box list_box">
		<view v-for="(item,i) in videoList" :key="i">
			<xgplayer :id='`myVideo${i}`' :videoData="item"></xgplayer>
			<text class="text">{{item.mpName}}</text>
		</view>
	</view>
</template>
<!-- <script src="//unpkg.byted-static.com/xgplayer/2.31.2/browser/index.js" type="text/javascript"></script> -->
<script>
	import xgplayer from './xgplayer.vue'
	export default {
		components: { xgplayer },
		data(){
			return {
				videoList: [{
					mpName: '测试视频',
					videourl: 'https://wethink-zs-180.cyberwisdom.net.cn/resource/resource/3052/1728525211024.mp4'
				}], // 视频列表
			}
		}
	}
</script>
