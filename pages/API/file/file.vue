<template>
	<!-- 上传组件（多媒体）: 多媒体包含图片、视频、文档等，封装能传参确认能上传范围比如type：
		【pic,doc,video】类似，尽量保证多场景使用；上传要有对应的默认成功失败回调但能在父组件定义回调方法 -->
	<view class="content">
		<view class="padding">
			<view>已选择文件列表：</view>
			<view v-for="(item,index) in files.values()" :key="index">
				<image style="width: 100rpx;height: 100rpx;" :src="item.path" mode="widthFix"></image>
				<view>文件名称：{{ item.name }}</view>
				<view style="margin-left: 10rpx;">大小：{{ item.size }}</view>
				<view style="margin-left: 10rpx;">状态：{{ item.type }}</view>
				<view style="margin-left: 10rpx;">进度：{{ item.progress }}</view>
				<view v-if="item.responseText" >服务端返回演示：{{ item.responseText }}</view>
				<view v-if="item.type=='fail'" @click="resetUpload(item.name)">重新上传</view>
				<view @click="clear(item.name)">删除</view>
			</view>
			
		</view>
		
		<view class="padding">
			<lsjUpload ref="lsjUpload" childId="upload1"
				width="180px" height="180px"
				:option="option"
				:toBase="false"
				:size="50"
				:count="1" formats=""
				:multiple="false"
				:debug="true" :instantly="true" :distinct="false"
				@change="onChange"
				@progress="onprogress"
				@uploadEnd="onuploadEnd"
				@permissionBefore="permissionBefore"
				@permissionFail="onpermissionFail">
				<view class="btn" :style="{width: '180px',height: '180px'}">选择附件</view>
			</lsjUpload>
			
			<view style="margin-top: 20rpx;">
				<view class="btn" style="width: 100px;height: 100rpx;" @click="upload">手动上传</view>
			</view>
			
		</view>
	</view>
</template>

<script>
	import lsjUpload from "../../../uni_modules/lsj-upload/components/lsj-upload/lsj-upload.vue"
export default {
	components: {lsjUpload},
	data() {
		return {
			// 上传接口参数
			option: {
				// 上传服务器地址，需要替换为你的接口地址
				url: 'http://hl.jw.com/dropbox/document/upload', // 该地址非真实路径，需替换为你项目自己的接口地址
				// 上传附件的key
				name: 'file',
				// 根据你接口需求自定义请求头,默认不要写content-type,让浏览器自适配
				header: {
					// 示例参数可删除
					'Authorization': 'bearer aa',
					'uid': '27',
					'accountid': '27'
				},
				// 根据你接口需求自定义body参数
				formData: { 
					// 'orderId': 1000
				}
			},
			// 文件回显列表
			files: new Map()
		}
	},
	onReady() {
		setTimeout(()=>{
			// console.log('----------------------演示动态更新参数 ==> setData-option的值---------------------');
			// this.$refs.lsjUpload.setData('formData.orderId','动态设置的参数'); 
			// 修改option对象的name属性
			// this.$refs.lsjUpload.setData('name','myFile');
			
			// console.log('----------------------演示初始化值，用于修改时 再次编辑时 需带入已上传文件---------------------');
			// 方式1 =====> 传入数组
			// let files1 = [{name: '1.png'},{name: '2.png',}];
			// 方式2 =====> 传入Map对象
			// let files2 = new Map();
			// files2.set('1.png',{name: '1.png'})
			// 此处调用setFiles设置初始files
			// this.$refs.lsjUpload.setFiles(files1);
			
			// ===========初始化界面===========
			// APP端因为是webview，层级比view高，此时若不希望点击触发选择文件，需要手动调用hide()
			uni.$emit('$upload-hide',{}); // 手动调用hide，隐藏所有的控件（透明层）
			this.$nextTick(()=>{
				this.$refs['lsjUpload'].show(); // 需要调用show()才能恢复覆盖层的点击(显示允许点击的控件（透明层）)
			})
		},2000)
	},
	methods: {
		// APP端 isPermissionInToast=false时，点击按钮检测到某项权限未授权时触发
		// 注意：不建议isPermissionInToast传false!!
		// 因为isPermissionInToast等于false时弹出的授权窗口用户点击同意后不会继续打开系统文件选择界面，需要用户再点一次选择文件按钮
		permissionBefore({permission,message},childId) {
			console.log(childId,'=============授权询问=============', permission,message);
		},
		// APP端 用户有权限点击不同意时触发
		// 当isPermissionBuiltInHandle=false不使用组件内置弹框引导用户开启时，可在此函数内自定义弹框
		onpermissionFail({permission, message, result}, childId) {
			console.log(childId,'====APP端 用户有权限点击不同意时触发=====', permission, message, result);
		},
		// 文件选择回调 (选择文件后触发，返回所有已选择文件Map集合)
		onChange(files, childId) {
			console.log('组件childId================', childId);
			console.log('当前选择的文件列表：***************',[...files.values()]);
			// console.log('当前选择的文件列表：***************',JSON.stringify([...files.values()]));
			// console.log('???????',uni.base64ToArrayBuffer([...files.values()][0].file));
			// 更新选择的文件 
			this.files = files;
			// 强制更新视图
			this.$forceUpdate();
			
			// Map对象for循环不显示，所以转成普通数组，如果你用不惯Map对象，也可以像这样转普通数组，组件使用Map主要是避免反复文件去重操作
			// let fileLists = [...this.files.values()];
			
			this.$nextTick(()=>{
				// 直接更新当前页面所有上传组件webview位置
				uni.$emit('$upload-show',{});
				// 以也可通过以下方式指定更新ref对应组件位置
				// this.$refs.lsjUpload0.show();
				// this.$refs.lsjUpload1.show();
				// this.$refs.lsjUpload2.show();
			});
			
		},
		// 上传进度回调(上传中持续触发，返回正在上传的文件对象，可通过set更新至页面显示上传进度)
		onprogress(item, childId) {
			// 更新当前状态变化的文件
			this.files.set(item.name, item);
			
			// console.log('打印对象',JSON.stringify(this.files.get(item.name)));
			console.log('打印对象', this.files.get(item.name));
			
			// Map对象for循环不显示，所以转成普通数组，如果你用不惯Map对象，也可以像这样转普通数组，组件使用Map主要是避免反复文件去重操作
			// let fileLists = [...this.files.values()];
			
			// 强制更新视图
			this.$forceUpdate();
			
		},
		// 某文件上传结束回调(成功失败都回调)
		// (上传结束回调，返回当前上传的文件对象，type等于success（上传成功）返回responseText属性（服务端返回数据）)
		onuploadEnd(item, childId) {
			// 这个item是哥File对象
			console.log(`${item.name} 已上传结束，上传状态= ${item.type}`, item);
			
			// 更新当前窗口状态变化的文件
			this.files.set(item.name, item);
			
			// ---可删除--演示上传完成后取服务端数据
			if (item['responseText']) {
				console.log('演示服务器返回的字符串JSON转Object对象');
				this.files.get(item.name).responseText = JSON.parse(item.responseText);
			}
			
			// Map对象for循环不显示，所以转成普通数组，如果你用不惯Map对象，也可以像这样转普通数组，组件使用Map主要是避免反复文件去重操作
			// let fileLists = [...this.files.values()];
			
			// 强制更新视图
			this.$forceUpdate();
			
			
			// ---可删除--演示判断是否所有文件均已上传成功
			let isAll = [...this.files.values()].find(item=>item.type!=='success');
			if (!isAll) {
				console.log('已全部上传完毕');
			} else {
				console.log(isAll.name+'待上传');
			}
			
		},
		// 手动上传
		upload() {
			// name=指定文件名，不指定则上传所有type等于waiting和fail的文件
			this.$refs['lsjUpload'].upload();
		},
		// 指定上传某个文件
		resetUpload(name) {
			this.$refs['lsjUpload'].upload(name);
		},
		// 移除某个文件
		clear(name) {
			// name=指定文件名，不传name默认移除所有文件
			this.$refs['lsjUpload'].clear(name);
		}
	}
}
</script>

<style>
	.header {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		height: 130rpx;
		line-height: 130rpx;
		text-align: center;
		font-size: 32rpx;
		background-color: #3F536E;
		color: #fff;
		z-index: 9999;
	}
	.content {
		padding-top: 130rpx;
	}
	.flex {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.inputs {
		height: 100rpx;
		border-bottom: 1rpx solid #808080;
	}
	.padding {
		padding-top: 30rpx;
		padding-left: 30rpx;
		padding-bottom: 50rpx;
	}
	.tab {display: flex;padding: 30rpx;}
	.tab view {height: 100rpx;line-height: 100rpx;flex: 1;text-align: center;border: 1rpx solid #2C405A;}
	.btn {
		height: 80rpx;
		background-color: #007AFF;
		color: #fff;
		border-radius: 10rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
	}
	.ts {
		line-height: 2em;
	}
	.nbtn {
		margin: 100rpx 50rpx;
	}
</style>
