<template>
    <!-- 上传组件（图片） ---   单独只能上传图片；能传参指定上传尺寸，要有默认上传尺寸；若是图片质量过大，需要进行压缩处理；上传要有对应的默认成功失败回调但能在父组件定义回调方法 -->
	<view>
		<view class="uni-common-mt">上传组件（图片）
         	<!-- <view :operation="compressedFile" :change:operation="uploadModule.readFileAsync"></view> -->
            <view v-if="showImgView" class="uni-list list-pd">
                <view class="uni-list-cell cell-pd">
                    <view class="uni-uploader">
                        <view class="uni-uploader-head">
                            <view class="uni-uploader-title">点击可预览选好的图片</view>
                            <view class="uni-uploader-info">{{imageList.length}}/{{ maxImgNum }}</view>
                        </view>
                        <view class="uni-uploader-body">
                            <view class="uni-uploader__files">
                                <block v-for="(image,index) in imageList" :key="index">
                                    <view class="uni-uploader__file">
                                        <image class="uni-uploader__img" :src="image" :data-src="image" @tap="previewImage"></image>
                                    </view>
                                </block>
                                <view class="uni-uploader__input-box">
                                    <view class="uni-uploader__input" @tap="chooseImage"></view>
                                </view>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
		</view>
	</view>
</template>
<script>
	import permision from "@/common/permission.js"
	var sourceType = [
		['camera'],
		['album'],
		['camera', 'album']
	]
	var sizeType = [
		['compressed'],
		['original'],
		['compressed', 'original']
	]
	export default {
        name: 'uploadImage',
        props: {
			/* 最多可以选择的图片张数，默认1 */
			maxImgNum: {
				type: Number,
				default: 1
			},
			/* 图片来源：0-拍照，1-相册，2-拍照或相册；默认2 */
			sourceTypeIndex: {
				type: Number,
				default: 2
			},
			/* 图片压缩还是原图：0-压缩，1-原图，2-压缩或原图；默认2 */
			sizeTypeIndex: {
				type: Number,
				default: 2
			},
			/* 单次可以选几张图片,是count的下标 单选/多选，默认8 */
			countIndex: {
				type: Number,
				default: 8
			},
			/* 最大上传尺寸，默认 2M */
			maxImageSize: {
				type: Number,
				default: 2
			},
            showImgView: {
                type: Boolean,
                default: true
            }
		},
		data() {
			return {
				imageList: [],
				sourceType: ['拍照', '相册', '拍照或相册'],
				sizeType: ['压缩', '原图', '压缩或原图'],
				count: [1, 2, 3, 4, 5, 6, 7, 8, 9],
                compressedFile: ''
			}
		},
		onUnload() {
			this.imageList = [],
            this.sourceType = ['拍照', '相册', '拍照或相册'],
            this.sizeType = ['压缩', '原图', '压缩或原图'];
		},
		methods: {
			chooseImage: async function() {
				let that = this
				// #ifdef APP-PLUS
				// TODO 选择相机或相册时 需要弹出 actionsheet ，目前无法获得是相机还是相册，在失败回调中处理
				if (this.sourceTypeIndex !== 2) { // 如果图片来源不是 “拍照或相册”；['拍照', '相册', '拍照或相册']
					let status = await this.checkPermission();
					if (status !== 1) {
						return;
					}
				}
				// #endif
                // 当最大图片数量不是1时，且图片满了后，弹出弹框确认是否需要清空
				if (this.imageList.length === this.maxImgNum && this.maxImgNum !== 1) {
					let isContinue = await this.isFullImg();
					console.log("是否继续?", isContinue);
					if (!isContinue) {
						return;
					}
				}
				uni.chooseImage({
					sourceType: sourceType[this.sourceTypeIndex],
					sizeType: sizeType[this.sizeTypeIndex],
					count: this.imageList.length + this.count[this.countIndex] > this.maxImgNum ? this.maxImgNum - this.imageList.length : this.count[this.countIndex],
					success: async (res) => {
                        let imgArr = []
                        console.log('res.tempFiles======', res.tempFiles);

                        // 1M = 1024 * 1024 b = 1024 kb
                        // 如果比设置的最大上传尺寸(默认2M)大,就return
                        for (let i = 0; i < res.tempFiles.length; i++) {
                            let size = res.tempFiles[i].size;
                            if (size > (this.maxImageSize * 1024 * 1024)) {
								uni.showModal({
									content: "上传图片的大小不能超过" + this.maxImageSize + "M",
									showCancel: false,
									success: (e) => {
									}
								})
                                console.log('上传图片的大小不能超过' + this.maxImageSize + 'M');
                                return false
                            }
                        }
                        // 本地路径 转为  在线url
                        for (let i = 0; i < res.tempFiles.length; i++) {
                            let item = res.tempFiles[i]
                            
                            // #ifdef APP-PLUS
                            console.log(i, '拿到的FIle对象=没压缩前===App=item===-', item);
							// console.log('判断是否是FIle对象=没压缩前====================', this.judgeIsFile(item));
                            // 压缩图片
							let comRes = that.compressedImg(item)
							console.log(i, '拿到的FIle对象==App==压缩后的））））））））））））', comRes);

							// let baseUrl = await this.readFileAsync(this.compressedFile);
							// console.log(i, '拿到的FIle对象==App==压缩后的=====-', baseUrl);
							imgArr.push(item.path)

                            // #endif
                            //  #ifdef H5
                            console.log(i, '拿到的FIle对象========-', item);
                            imgArr.push(item.path)
                            // #endif
                        }
                        if (this.maxImgNum === 1) {
                            this.imageList = imgArr;
                        } else {
                            this.imageList = this.imageList.concat(imgArr);
                        }
                        this.$emit('on-success', this.imageList)
					},
					fail: (err) => {
						console.log("err: ",err);
                        this.$emit('on-error', err)
						// #ifdef APP-PLUS
						if (err['code'] && err.code !== 0 && this.sourceTypeIndex === 2) {
							this.checkPermission(err.code);
						}
						// #endif
						// #ifdef MP
						if(err.errMsg.indexOf('cancel') !== '-1'){
							return;
						}
						uni.getSetting({
							success: (res) => {
								let authStatus = false;
								switch (this.sourceTypeIndex) {
									case 0:
										authStatus = res.authSetting['scope.camera'];
										break;
									case 1:
										authStatus = res.authSetting['scope.album'];
										break;
									case 2:
										authStatus = res.authSetting['scope.album'] && res.authSetting['scope.camera'];
										break;
									default:
										break;
								}
								if (!authStatus) {
									uni.showModal({
										title: '授权失败',
										content: 'Hello uni-app需要从您的相机或相册获取图片，请在设置界面打开相关权限',
										success: (res) => {
											if (res.confirm) {
												uni.openSetting()
											}
										}
									})
								}
							}
						})
						// #endif
					}
				})
			},
			judgeIsFile(object) {
				console.log(i, '进来了========-', object);
				// return object instanceof File || (obj && obj.constructor.name === 'File');
				// return object instanceof File && 
				// 	typeof object.name === 'string' && 
				// 	typeof object.size === 'number' && 
				// 	typeof object.type === 'string' && 
				// 	typeof object.slice === 'function';
			},
			compressedImg(item) {
				return new Promise((resolve) => {
					uni.compressImage({
                                src: item.path,
						quality: 20,
						success: async compressRes => {
							that.compressedFile = compressRes.tempFilePath; // 打印临时文件路径
							console.log('打印临时文件路径--------', that.compressedFile);
							
							// 如果需要将临时文件路径转换为File对象，可以使用如下方式：
							plus.io.resolveLocalFileSystemURL(that.compressedFile, function(entry) {
								console.log('转换后-***********************', entry.isFile);
								resolve(entry)
								// entry.file(function(file) {
								// 	console.log('转换后-***********************', file);
								// 	// console.log('判断是否是FIle对象=压缩后的==================', that.judgeIsFile(file));
								// 	// 转换为File对象
								// 	const imageFile = new plus.io.File(file.name, file.fullPath);
								// 	// 此时可以使用imageFile对象进行后续的上传操作
								// 	console.log(i, '拿到的FIle对象==App==压缩后的））））））））））））', imageFile);
								// });
							}, function(e) {
								console.log("读取文件失败：" + e.message);
							});

						}
					})
				})
			},
			isFullImg: function() {
				return new Promise((res) => {
					uni.showModal({
						content: "已经有" + this.maxImgNum + "张图片了,是否清空现有图片？",
						success: (e) => {
							if (e.confirm) {
								this.imageList = [];
								res(true);
							} else {
								res(false)
							}
						},
						fail: () => {
							res(false)
						}
					})
				})
			},
			previewImage: function(e) {
				var current = e.target.dataset.src
				uni.previewImage({
					current: current,
					urls: this.imageList
				})
			},
            // 查看是否授权
			async checkPermission(code) {
				let type = code ? code - 1 : this.sourceTypeIndex;
				let status = permision.isIOS ? await permision.requestIOS(sourceType[type][0]) :
					await permision.requestAndroid(type === 0 ? 'android.permission.CAMERA' :
						'android.permission.READ_EXTERNAL_STORAGE');
				if (status === null || status === 1) {
					status = 1;
				} else {
					uni.showModal({
						content: "没有开启权限",
						confirmText: "设置",
						success: function(res) {
							if (res.confirm) {
								permision.gotoAppSetting();
							}
						}
					})
				}
				return status;
			}
		}
	}
</script>
<script module="uploadModule" lang="renderjs">
    import VConsole from 'vconsole'
    new VConsole();  
	// 视图层
    export default{
		data(){
			return {
			}
		},
        mounted(){
		},
		onunload() {
		},
        methods:{
            async readFileAsync(filePath) {
                    try { // 增加这步,是为了解决异步返回时间不确定
                        const result = await this.convertToBase64(filePath);
                        return result
                    } catch (error) {
                        console.log(error);
                    }
            },
            // 图片格式转base64
            async convertToBase64(filePath) {
                if (!filePath) return
                return new Promise(async (resolve, reject) => {
                    
                    // 如果需要将临时文件路径转换为File对象，可以使用如下方式：

                    const response = await fetch(filePath);
                    console.log('response--------', response);
                    const blob = await response.blob();
                    const reader = new FileReader();
                    reader.onloadend = async () => {
                        const file = await this.base64ImgtoFile(reader.result);
                        console.log('file--------', file);
                        resolve(file)
                    };
                    reader.readAsDataURL(blob);
                });
            },
            base64ToBlob(base64, type) {
                const byteCharacters = atob(base64.split(',')[1]);
                const byteNumbers = new Array(byteCharacters.length);
                for (let i = 0; i < byteCharacters.length; i++) {
                    byteNumbers[i] = byteCharacters.charCodeAt(i);
                }
                const byteArray = new Uint8Array(byteNumbers);
                return new Blob([byteArray], { type: type });
            },
            base64ToArrayBuffer(base64) {
                var binary_string = atob(base64);
                var len = binary_string.length;
                var bytes = new Uint8Array(len);
                for (var i = 0; i < len; i++) {
                    bytes[i] = binary_string.charCodeAt(i);
                }
                return bytes.buffer;
            },
            // base64转file
            base64ImgtoFile(srcBase64, filename = 'file') {
                //将base64格式分割：['data:image/png;base64','XXXX']
                const arr = srcBase64.split(',');
                // .*？ 表示匹配任意字符到下一个符合条件的字符 刚好匹配到：
                // image/png
                const mime = arr[0].match(/:(.*?);/)[1]; //image/png
                //[image,png] 获取图片类型后缀
                const suffix = mime.split('/')[1]; //png
                const bstr = atob(arr[1]); //atob() 方法用于解码使用 base-64 编码的字符串
                let n = bstr.length;
                const u8arr = new Uint8Array(n);
                while (n--) {
                    u8arr[n] = bstr.charCodeAt(n);
                }
                return new File([u8arr], `${filename}.${suffix}`, {
                    type: mime
                });
            },
        }
    }
</script>

<style>
	.cell-pd {
		padding: 22rpx 30rpx;
	}

	.list-pd {
		margin-top: 50rpx;
	}
</style>
