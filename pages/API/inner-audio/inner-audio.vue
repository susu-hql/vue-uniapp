<template>
	<view class="uni-padding-wrap">
		<page-head title="audio"></page-head>
		<!-- https://ext.dcloud.net.cn/plugin?id=12417 -->
		<le-audio ref="le-audio"
			:audioUrl="fileUrl" 
			:showAudioSpeedIcon="true"
		></le-audio>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				fileUrl: ''
			}
		},
		computed: {
		},
		onLoad() {
			setTimeout(()=>{
				this.fileUrl = 'https://wethink-zs-180.cyberwisdom.net.cn/resource/resource/4203/c77020c0d00d755877ac6739d838abb7.mp3';
			},1000)
		},
		onUnload() {
		},
		onHide: function() { // 退出应用时调用
			this.$refs['le-audio'].beforeLeave()
		},
		beforeDestroy() { // 点击返回时调用
			this.$refs['le-audio'].beforeLeave()
		},
		methods: {
		}
	}
</script>
<style>
.content {
	padding-top: 200rpx;
}
</style>
