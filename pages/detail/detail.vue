<template>
	<view>
		<!-- 信息 -->
		<detail-info :item="detail"></detail-info>
		<!-- 评论 -->
		<view class="u-comment-title">
			最新评论 {{comment.count}}
		</view>
		<view class="uni-comment u-comment">
			<block v-for="(item,index) in comment.list" :key="index">
				<comment-list :item="item" :index="index"></comment-list>
			</block>
		</view>
		<!-- 占位盒子 -->
		<view style="height: 120rpx;"></view>
		<!-- 输入框 -->
		<user-chat-bottom @submit="submit"></user-chat-bottom>
		<!-- 分享 -->
		<more-share :shareShow="shareShow" :sharedata="sharedata" @toggolShow="toggolShow"></more-share>
	</view>
</template>
 
<script>
import detailInfo from '@/components/detail/detail-info.vue'
import commentList from '@/components/detail/comment-list.vue'
import userChatBottom from '@/components/user-chat/user-chat-bottom.vue'
import moreShare from '@/components/common/more-share.vue'
import time from '@/common/time.js'

export default {
	data() {
		return {
			shareShow: false,
			sharedata: {
				title:"",
				url:"",
				titlepic:"",
				shareType:0,
			},
			detail: {
				userpic:"",
				username:"",
				sex:0, //0 男 1 女
				age:0,
				content:"",
				isguanzhu:false,
				title:"",
				titlepic:"",
				morepic:[],
				video:false,
				share:false,
				path:"",
				sharenum:0,
				commentnum:0,
				goodnum:0,
				creat_time:0
			},
			comment: {
				count: 0,
				list: []
			}
		}
	},
	onLoad(e) {
		this.initData(JSON.parse(e.detailData))	
	},
	// 监听导航右边按钮
	onNavigationBarButtonTap(e) {
		if (e.index == 0) {
			this.toggolShow();
		}
	},
	components: {
		detailInfo,
		commentList,
		userChatBottom,
		moreShare
	},
	methods: {
		// 初始化分享内容
		__initShare(obj){
			this.sharedata = {
				title:obj.title,
				content:obj.title,
				url:"http://dcloud.io/",
				titlepic:obj.titlepic ? obj.titlepic : 'https://img-cdn-qiniu.dcloud.net.cn/uniapp/app/share-logo@3.png',
				shareType:0,
			}
		},
		// 初始化数据
		initData (obj) {
			// 修改窗口标题
			uni.setNavigationBarTitle({ title: obj.title })
			// 加载分享内容
			this.__initShare(obj);
			// 加载中
			uni.showLoading({ title: 'Loading...',mask:true });
			
			obj.morepic = [];
			obj.content = "";
			obj.goodnum = obj.infonum.dingnum;
			this.detail = obj;
			this.comment.count = obj.commentnum;
			// 获取文章详情
			this.getdetail();
			if (this.comment.count) {
				this.getcomment();
			}
		},
		updateData(data){
			switch (data.type){
				case "guanzhu":
				this.updateGuanZhu(data);
					break;
				case "support":
				this.updateSupport(data);
					break;
			}
		},
		updateGuanZhu(data){
			this.detail.isguanzhu = data.data;
		},
		updateSupport(data){
			if (data.do == 'smile') {
				this.detail.infonum.index = 1;
				this.detail.goodnum++;
			}
		},
		async getdetail () {
			let [err,res] =await this.$http.get('/post/'+this.detail.id);
			let error = this.$http.errorCheck(err,res,()=>{
				uni.hideLoading();
			},()=>{
				uni.hideLoading();
			});
			if (!error) return;
			
			let data = res.data.data.detail;
			this.detail.content = data.content;
			
			let images = [];
			for (let i = 0; i < data.images.length; i++) {
				images.push(data.images[i].url);
			}
			this.detail.morepic = images;
			this.detail.age = data.user.userinfo.age;
			this.detail.sex = data.user.userinfo.sex;
			this.detail.creat_time = data.creat_time;
			// console.log(this.detail);
			return uni.hideLoading();
		},
		// 获取评论
		async getcomment(){
			let [err,res] = await this.$http.get('/post/'+this.detail.id+'/comment');

			if (!this.$http.errorCheck(err,res)) return;
			let list = res.data.data.list;
			this.comment.list = this.comment.list.concat(this.__ArrSort(list));
		},
		__ArrSort(arr,id = 0){
			var temp = [],lev=0;
			// console.log(JSON.stringify(arr))
			var forFn = function(arr, id,lev){
				for (var i = 0; i < arr.length; i++) {
					var item = arr[i];
					if (item.fid==id) {
						item.lev=lev;
						temp.push({
							id:item.id,
							fid:item.fid,
							userid:item.user.id,
							userpic:item.user.userpic,
							username:item.user.username,
							time:time.gettime.gettime(item.create_time),
							data:item.data,
						});
						forFn(arr,item.id,lev+1);
					} 
				}
			};
			forFn(arr, id,lev);
			return temp;
		},
		submit (data) {
			console.log(data);
			let obj = {
				id: 1,
				level: 0,
				userPic: 'https://img-cdn-qiniu.dcloud.net.cn/uniapp/images/uni@2x.png',
				userName: '小麦',
				time: time.gettime.gettime(new Date().getTime()),
				data: data
			}
			
			this.comment.list.push(obj)
		},
		toggolShow () {
			this.shareShow = !this.shareShow
		},
		reply(id){
			this.reply_id = id;
			this.focus = true;
		},
		blur(){
			this.focus = false;
			this.reply_id = 0;
		},
	}
}
</script>

<style lang="scss" scoped>
.u-comment-title {
	padding: 20rpx;
	font-size: 30rpx;
	font-weight: bold;
}
.u-comment {
	padding: 0 20rpx;
}

</style>
