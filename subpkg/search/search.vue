<template>
	<view>
		<view class="search-box">
			<!-- 使用 uni-ui 提供的搜索组件 -->
			<uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		</view>
		<!-- 搜索建议列表 -->
		<view class="sugg-list">
			<view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
				<view class="goods-name">{{ item.goods_name }}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			// 延时器的timeID
			timer: null,
			//搜索关键词
			kw: '',
			// 搜索结果列表
			searchResults: []
		};
	},
	methods: {
		input(e) {
			// 清楚timer
			clearTimeout(this.timer);
			// 重新启动一个延时器 并把timerid 赋值给this.timer
			this.timer = setTimeout(() => {
				// 如果500毫秒内，没有触发新的输入事件，则为搜索关键词赋值
				this.kw = e;
				// 根据关键词，查询搜索建议列表
				this.getSearchList();
				console.log(this.kw);
			}, 500);
		},
		// 根据搜索关键词，搜索商品建议列表
		// 根据搜索关键词，搜索商品建议列表
		async getSearchList() {
			// 判断关键词是否为空
			if (this.kw === '') {
				this.searchResults = [];
				return;
			}
			// 发起请求，获取搜索建议列表
			const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: this.kw });
			if (res.meta.status !== 200) return uni.$showMsg();
			this.searchResults = res.message;
			console.log(this.searchResults);
		},
		gotoDetail(item) {
			console.log(item.goods_id);
			uni.navigateTo({
				url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
			});
		}
	}
};
</script>

<style lang="scss">
.uni-searchbar {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	position: relative;
	padding: 16rpx;
	/* 将默认的 #FFFFFF 改为 #C00000 */
	background-color: #c00000 !important;
}
.search-box {
	z-index: 999;
	position: initial;
	top: 0;
}
.sugg-list {
	padding: 0 5px;

	.sugg-item {
		font-size: 12px;
		padding: 13px 0;
		border-bottom: 1px solid #efefef;
		display: flex;
		align-items: center;
		justify-content: space-between;

		.goods-name {
			// 文字不允许换行（单行文本）
			white-space: nowrap;
			// 溢出部分隐藏
			overflow: hidden;
			// 文本溢出后，使用 ... 代替
			text-overflow: ellipsis;
			margin-right: 3px;
		}
	}
}
</style>
