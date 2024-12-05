<template>
	<view>
		<page-head :title="title"></page-head>
		<view class="uni-common-mt">
            <view style="height: 100px;line-height: 100px;" @click="chooseAvatar()">
                <text>头像 </text>
                <text>
					<image style="width: 100px; height: 100px; border: 1px solid red;"
						mode="aspectFill" :src="userPoto"></image>
				</text>
            </view>
			<uploadImage ref="uploadImg" :maxImgNum="3" @on-success="savaUrl" @on-error="errHandle" :showImgView="false"></uploadImage>
		</view>
	</view>
</template>
<script>
	import uploadImage from "./upload-image.vue"
	export default {
		data() {
			return {
				title: 'choose/previewImage',
				userPoto: ''
			}
		},
		components: {uploadImage},
		onUnload() {
		},
		methods: {
			chooseAvatar() {
				// 这里是调用子组件的方法：打开上传图片
				this.$refs['uploadImg'].chooseImage()
			},
			savaUrl(url) {
				console.log('image----应该拿到上传成功后的url', url);
				this.userPoto = url[0]
			},
			errHandle(error) {
				console.log('上传失败回调', error);
			}
		}
	}
</script>

<style>
</style>
