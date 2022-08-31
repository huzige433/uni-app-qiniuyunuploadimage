<template>
	<view class="content">
		<view class="uni-margin-wrap">
			<swiper class="swiper" circular indicator-dots="indicatorDots" autoplay="autoplay" interval=3000
				:duration="duration" :style="{ height: '500px' }">
				<swiper-item v-for="(item,index) in imgurls" :key="index">
					<view class="swiper-item uni-bg-red">
						<img :src="item.url" alt="" srcset="">
					</view>
				</swiper-item>
			</swiper>
		</view>
		
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
				imgurls:[],
				collection:uniCloud.database().collection('imagedb')
			}
		},
		onLoad() {
			console.log(this.imgurls)
			var that =this
			this.collection.limit(5).get().then((res)=>{
				console.log(res.result.data)
				that.imgurls=res.result.data
			})
		},
		methods: {
			async updataimage() {
				var that =this
				const todo = uniCloud.importObject('qiniufunction')
				const upToken = await todo.uptoken()
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
								'token': upToken
							},
							success: (uploadFileRes) => {
								let key = JSON.parse(uploadFileRes.data).key;
								let partImgUrl = "http://rhgwdl6lb.hn-bkt.clouddn.com/"+key;
								console.log(partImgUrl)
								uni.showToast({
									title:partImgUrl,
									duration:3000,
								})
								uni.hideLoading()
								that.collection.add([{
								  name: key,
								  url:partImgUrl
								}]).then((res)=>{console.log("新增成功")})
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
	
	.uni-margin-wrap {
			width: 690rpx;
			width: 100%;
		}
		.swiper {
			height: 300rpx;
		}
		.swiper-item {
			display: block;
			height: 300rpx;
			line-height: 300rpx;
			text-align: center;
		}
		.swiper-list {
			margin-top: 40rpx;
			margin-bottom: 0;
		}
		.uni-common-mt {
			margin-top: 60rpx;
			position: relative;
		}
		.info {
			position: absolute;
			right: 20rpx;
		}
		.uni-padding-wrap {
			width: 550rpx;
			padding: 0 100rpx;
		}
</style>
