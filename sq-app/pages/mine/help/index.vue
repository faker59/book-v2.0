<template>
	<view class="help-container">
		<view v-for="(item, findex) in list" :key="findex" :title="item.title" class="list-title">
			<view class="text-title">
				<view :class="item.icon"></view>{{ item.title }}
			</view>
			<view class="childList">
				<view v-for="(child, zindex) in item.childList" :key="zindex" class="question" hover-class="hover"
					@click="handleText(child)">
					<view class="text-item">{{ child.title }}</view>
					<view class="line" v-if="zindex !== item.childList.length - 1"></view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [{
						icon: 'iconfont icon-github',
						title: '拾柒爱阅读问题',
						childList: [{
							title: '拾柒爱阅读可以做些什么？',
							content: '如果你停止，就是谷底。如果你还在继续，就是上坡。这是我听过关于人生低谷最好的阐述。借此我做了一个在线阅读平台， 使现代人可以利用碎片化来阅读。并且设计加入了社区功能，使得用户可以一起讨论阅读心得，并发表在社区里，产生一个生态闭环，实现用户快速裂变。'
						}, {
							title: '拾柒爱阅读开源吗？',
							content: '开源'
						}, {
							title: '拾柒爱阅读可以商用吗？',
							content: '可以'
						}, {
							title: '拾柒爱阅读官网地址多少？',
							content: 'https://gitee.com/cxq21/book'
						}, {
							title: '拾柒爱阅读文档地址多少？',
							content: 'https://gitee.com/cxq21/book'
						}]
					},
					{
						icon: 'iconfont icon-help',
						title: '其他问题',
						childList: [{
							title: '如何退出登录？',
							content: '请点击[我的] - [应用设置] - [退出登录]即可退出登录',
						}, {
							title: '如何修改用户头像？',
							content: '请点击[我的] - [选择头像] - [点击提交]即可更换用户头像',
						}, {
							title: '如何修改登录密码？',
							content: '请点击[我的] - [应用设置] - [修改密码]即可修改登录密码',
						},{
							title: '如何反馈问题？',
							content: '请登录电脑端，发布反馈问题话题，工作人员会在工作日回复！',
						}]
					}
				]
			}
		},
		methods: {
			handleText(item) {
				this.$tab.navigateTo(`/pages/common/textview/index?title=${item.title}&content=${item.content}`)
			}
		}
	}
</script>

<style lang="scss" scoped>
	page {
		background-color: #f8f8f8;
	}

	.help-container {
		margin-bottom: 100rpx;
		padding: 30rpx;
	}

	.list-title {
		margin-bottom: 30rpx;
	}

	.childList {
		background: #ffffff;
		box-shadow: 0px 0px 10rpx rgba(193, 193, 193, 0.2);
		border-radius: 16rpx;
		margin-top: 10rpx;
	}

	.line {
		width: 100%;
		height: 1rpx;
		background-color: #F5F5F5;
	}

	.text-title {
		color: #303133;
		font-size: 32rpx;
		font-weight: bold;
		margin-left: 10rpx;

		.iconfont {
			font-size: 16px;
			margin-right: 10rpx;
		}
	}

	.text-item {
		font-size: 28rpx;
		padding: 24rpx;
	}

	.question {
		color: #606266;
		font-size: 28rpx;
	}
</style>
