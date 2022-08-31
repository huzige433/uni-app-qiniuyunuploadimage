<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
			<u-search placeholder="日照香炉生紫烟" v-model="keyword" :clearabled="true"
				:actionStyle="{color:'red',border:'5px solid red'}"></u-search>
		</view>
		<button @click="updataimage">test</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				keyword: '',
				upToken: ''
			}
		},
		onLoad() {

			const todo = uniCloud.importObject('qiniufunction')
			const res = todo.uptoken()
			res.then(
				(value) => {
					this.upToken = value
					console.log(this.upToken)
				},
				(rezon) => {
					console.log(rezon)
				}
			)
		},
		methods: {
			updataimage() {
				var that = this
				uni.chooseImage({
					count: 1, //默认9
					sizeType: ["original", "compressed"], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ["album", "camera"], //从相册选择
					success(chooseImageRes) {
						uni.showLoading({
							title: "文件上传中",
							mask: true
						});
						const tempFilePaths = chooseImageRes.tempFilePaths;
						uni.uploadFile({
							url: 'https://up-z2.qiniup.com',
							filePath: tempFilePaths[0],
							name: 'file',
							formData: {
								'key': that.guid()+".jpg",
								'token': that.upToken
							},
							success: (uploadFileRes) => {
								let key = JSON.parse(uploadFileRes.data).key;
								let partImgUrl = "http://rhgwdl6lb.hn-bkt.clouddn.com/"+key;
								console.log(partImgUrl)
								uni.hideLoading()
							},
							fail: (err) => {
								console.log('fail', err);
								uni.hideLoading()
							}

						})

					}
				})
			},
			guid() {
				    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
				        var r = Math.random() * 16 | 0,
				            v = c == 'x' ? r : (r & 0x3 | 0x8);
				        return v.toString(16);
				    });
			
			},
		},
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
