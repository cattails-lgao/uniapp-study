<template>
	<view class="index-list animated fadeIn fast">
		<view class="index-list-one flex flex-item flex-JustBetween">
			<view class="flex flex-item">
				<image :src="item.userpic" lazy-load></image>
				{{item.username}}
			</view>
			<view
				class="flex flex-item"
				v-show="!item.isguanzhu"
				@tap="follow">
				<view class="icon iconfont icon-zengjia1"></view>
				关注
			</view>
		</view>
		<view class="index-list-two" @tap="openDetail">{{item.title}}</view>
		<view class="index-list-three flex flex-item flex-JustCenter" @tap="openDetail">
			<!-- 图片 -->
			<template v-if="item.titlepic">
				<image :src="item.titlepic" mode="widthFix" lazy-load></image>
			</template>
			<!-- 视频 -->
			<template v-if="item.type === 'video'">
				<view class="index-list-play icon iconfont icon-bofang"></view>
				<view class="index-list-info">
					{{item.playnum}} 次播放 {{item.longtime}}
				</view>	
			</template>				
		</view>
		<view class="index-list-four flex flex-item flex-JustBetween">
			<view class="flex flex-item">
				<view class="flex flex-item" @tap="operate('smile')">
					<view :class="['icon', 'iconfont', 'icon-xiaolian1', {'active': item.infonum.index === 1}]"></view>
					{{item.infonum.dingnum}}
				</view>
				<view class="flex flex-item" @tap="operate('cry')">
					<view :class="['icon', 'iconfont', 'icon-kulian', {'active': item.infonum.index === 2}]"></view>
					{{item.infonum.cainum}}
				</view>
			</view>
			<view class="flex flex-item">
				<view class="flex flex-item">
					<view class="icon iconfont icon-pinglun1"></view>
					{{item.commentnum}}
				</view>
				<view class="flex flex-item">
					<view class="icon iconfont icon-zhuanfa"></view>
					{{item.sharenum}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			item: Object,
			index: {
				type: Number,
				default: 0
			},
			index1: {
				type: Number,
				default: 0
			}
		},
		data() {
			return {
				
			};
		},
		methods: {
			// 关注
			async follow () {
				try{
					let [err,res] = await this.$http.post('/follow',{
						follow_id:this.item.userid
					},{
						token:true,
						checkToken:true,
						checkAuth:true
					});
					// 错误处理
					if (!this.$http.errorCheck(err,res)){
						return;
					}
					// 通知首页修改数据
					uni.showToast({ title: '关注成功' });
					let resdata = {
					 	type:"guanzhu",
					 	userid:this.item.userid,
					 	data:true
					};
					// 通知父组件
					this.$emit('changeEvent',resdata);
					// 通知首页
					uni.$emit('updateData',resdata);
				}catch(e){ return; }
			},
			// 赞 踩
			async operate (type) {
				try{
					let index = (type === 'smile') ? 1 : 2; // 操作后的状态
					if(this.item.infonum.index === index) return; // 状态相同不修改
					let [err,res] = await this.$http.post('/support',{
						post_id:this.item.id,
						type:index-1
					},{
						token:true,
						checkToken:true,
						checkAuth:true
					});
					// 错误处理
					if (!this.$http.errorCheck(err,res)) return;
					uni.showToast({
						title: index == 1 ? "顶成功" : "踩成功"
					});
					let resdata = {
						type:"support",
						post_id:this.item.id,
						do:type
					};
					// 通知父组件
					this.$emit('changeevent',resdata);
					// 通知全局
					uni.$emit("updateData",resdata);
				}catch(e){ return; }
			},
			// 进入详情页
			openDetail () {
				uni.navigateTo({
					url:'../../pages/detail/detail?detailData='+JSON.stringify(this.item)
				})
			}
		}
	}
</script>

<style scoped lang="scss">
.index-list{
	padding: 20rpx;
	border-bottom: 1rpx solid #efefef;
	
	.index-list-one {
		&>view:first-of-type {
			color: #999;
			image {
				width: 85rpx;
				height: 85rpx;
				border-radius: 100%;
				margin-right: 10rpx;
			}
		}
		&>view:last-of-type {
			background: #f4f4f4;
			border-radius: 5rpx;
			padding:0 10rpx;
			padding-right: 15rpx;
		}
	}
	.index-list-padding,.index-list-two,.index-list-three,.index-list-four {
		padding-top: 15rpx;
	}
	.index-list-two {
		font-size: 32rpx;
	}
	.index-list-three {
		position: relative;
		image {
			width: 100%;
			border-radius: 20rpx;
		}
		.index-list-play {
			position: absolute;
			font-size: 100rpx;
			color: #fff;
		}
		.index-list-info {
			position: absolute;
			background: rgba(0,0,0,.5);
			border-radius: 40rpx;
			color:#fff;
			font-size: 22rpx;
			padding: 0 15rpx;
			bottom: 8rpx;
			right: 8rpx;
		}
	}
	.index-list-four {
		&>view {
			&>view {
				color: #ccc;
				&>view {
					margin-right: 15rpx;
				}
			}
			&>view:last-of-type {
				margin-left: 15rpx;
			}
		}
		.active {
			color: lightblue;
		}
	}
}
</style>
