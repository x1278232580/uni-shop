<template>
	<view>
		<view class="goods-list">
			<view v-for="(item, i) in goodsList" :key="i" @click="gotoDetail(item)">
				<!-- 为组件动态绑定goods属性的值 -->
				<my-goods :goods="item"></my-goods>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			// 请求参数对象
			queryObj: {
				// 查询关键字
				query: '',
				// 商品分类ID
				cid: '',
				// 页码值
				pagenum: 1,
				// 每页显示多少数据
				pagesize: 10
			},
			// 商品列表的数据
			goodsList: [],
			// 总数量，用来实现分页
			total: 0,
			// 默认的空图片
			defaultPic: 'https://img3.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png',
			// 是否正在请求数据
			isloading: false
		};
	},
	onLoad(options) {
		// 将页面参数转存到 this.queryObj对象中
		this.queryObj.query = options.query || '';
		this.queryObj.cid = options.cid || '';
		// 调用获取商品列表数据的方法
		this.getGoodsList();
	},
	methods: {
		// 获取商品列表的数据
		async getGoodsList(cd) {
			// 打开节流阀
			this.isloading = true;
			// 发起请求
			const { data: res } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj);
			// 关闭节流阀
			this.isloading = false;
			// 只要数据请求完毕，就立即按需调用cd回调函数
			cd && cd();
			// 判断是否请求
			if (res.meta.status !== 200) return uni.$showMsg();
			// 为数据赋值
			this.goodsList = [...this.goodsList, ...res.message.goods];
			this.total = res.message.total;
			console.log(this.goodsList);
			console.log(this.total);
		},
		// 点击跳转到商品详情页面
		gotoDetail(item) {
			uni.navigateTo({
				url: '/subpkg/goods_detail/goods_detail?goods_id' + item.goods_id
			});
		}
	},
	// 触底的事件
	onReachBottom() {
		// 判断是否还有下一页的数据   当前的页码值 * 每页显示多少条数据 >= 总数条数
		if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕');
		// 判断是否正在请求其他数据，如果是，则不发起额外的请求
		if (this.isloading) return;
		// 让页码值自增 +1
		this.queryObj.pagenum += 1;
		// 重新获取列表数据
		this.getGoodsList();
	},
	// 下拉刷新的事件
	onPullDownRefresh() {
		// 1.重置关键数据
		this.queryObj.pagenum = 1;
		this.total = 0;
		this.isloading = false;
		this.goodsList = [];
		// 2.重新发起请求
		this.getGoodsList(() => uni.stopPullDownRefresh());
	}
};
</script>

<style lang="scss">
.goods-item {
	display: flex;
	padding: 10px 5px;
	border-bottom: 1px solid #f0f0f0;

	.goods-item-left {
		margin-right: 5px;

		.goods-pic {
			width: 100px;
			height: 100px;
			display: block;
		}
	}

	.goods-item-right {
		display: flex;
		flex-direction: column;
		justify-content: space-between;

		.goods-name {
			font-size: 13px;
		}

		.goods-price {
			font-size: 16px;
			color: #c00000;
		}
	}
}
</style>
