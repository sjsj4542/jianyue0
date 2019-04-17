<template>
	<view class="container">
		<view class="leftright">
			<view class="left">
				<image src="../../static/page.png" />
			</view>
			<view class="right">
				<button class="green-btn" @tap="postArticle">发布</button>
			</view>
		</view>
		<!-- 文章标题输入框，和title变量绑定 -->
		<input type="text" v-model="title" placeholder="请输入标题" class="card1" />
		<!-- 文章内容输入的多行文本域，绑定content变量 -->
		<textarea placeholder="输入内容" v-model="content" class="content" />
		<!-- 选择图片的按钮，点击触发chooseImg方法 -->
		<button class="add-btn" @tap="chooseImg">添加图片</button>
		<text>预览</text>
		<!-- 使用graceUI的富文本解析功能，来预览文章内容 -->
		<view class="grace-text">
				<rich-text :nodes="content" bindtap="tap"></rich-text>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			title: '',
			content: '',
			content1:'',
			userId: uni.getStorageSync('login_key').userId,
			imgs: []
		};
	},
	onLoad() {},
	methods: {
		chooseImg: function() {
			var _this = this;
			uni.chooseImage({
				count: 1,
				sizeType: ['original', 'compressed'],
				sourceType: ['album'],
				success: function(res) {
					console.log(JSON.stringify(res.tempFilePaths));
					uni.uploadFile({
						url: _this.apiServer + '/avatar/upload',
						filePath: res.tempFilePaths[0],
						name: 'file',
						success: uploadFileRes => {
							//图片上传成功，回显图片地址
							console.log(uploadFileRes.data);
							//将图片地址加入imgs数组
							_this.imgs.push(uploadFileRes.data);
							//将图片地址拼接HTML标签，加入文章内容
							_this.content += '<img src="' + uploadFileRes.data + '" width = "100%"/>';
							console.log("上传的文章源码："+_this.content);
						}
					});
				}
			});
		},
		postArticle: function() {
			var _this = this;
			uni.request({
				url: this.apiServer + '/article/add',
				method: 'POST',
				header: { 'content-type': 'application/x-www-form-urlencoded' },
				data: {
					uId: _this.userId,
					title: _this.title,
					content: '<div>' + _this.content1 + '</div>'
				},
				success: res => {
					if (res.data.code === 0) {
						//获得发布文章成功返回的文章id
						var aId = res.data.data;
						console.log(aId);
						uni.showToast({
							title: '发布成功'
						});
						//将文章id和文章对应的图片地址数组传到后台，存入数据库
						uni.request({
							url: this.apiServer + '/img/add',
							method: 'POST',
							header: { 'content-type': 'application/x-www-form-urlencoded' },
							data: {
								aId: aId,
								imgs: JSON.stringify(_this.imgs)  //序列化imgs数组
							},
							success: res => {
								if (res.data.code === 0) {
									console.log('文章图片地址已写入数据库');
								}
							}
						});
						uni.switchTab({
							url: '../index/index'
						});
					}
				}
			});
		}
	}
};
</script>

<style>
.leftright{
	display: flex;
	justify-content: space-between;
	margin-top: 10upx;
}	
.right{

}
.card1{
	border: 1upx solid #E9E9E9;
	height: 80upx;
}
/* 圆形按钮 */
.btn-floating{
	/* 阴影效果，四个参数分别是：水平阴影位置、垂直阴影位置、阴影尺寸位置大小、阴影颜色 */
	box-shadow: 2px 5px 10px #AAA;
	width: 10px;
	height: 20px;
	margin: 10px;
	border-radius: 50%;
	padding: 0;
	cursor: pointer;
	border: none;
}
/* 绿色渐变色按钮 */
.greed-gradient{
	background: linear-gradient(40deg, #AFEEEE, #008080);
	color: #FFF;
}
.logo {
	display: flex;
	justify-content: space-between;
}
.content {
	margin-top: 10px;
	border: 1upx solid #E9E9E9;
}

.title {
	height: 50px;
}
.left image{
	width: 80upx;
	height: 80upx;
	font-size: 15upx;
}

.add-btn {
	margin-top: 10px;
	margin-bottom: 10px;
}
.green-btn{
	display: flex;
	justify-content: center;
	align-items: center;
	width: 150upx;
	height: 75upx;
	font-size: 35upx;
}
</style>
