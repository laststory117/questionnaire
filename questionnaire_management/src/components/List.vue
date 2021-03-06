<template>
	<div class="container">
		<div class="add-wrapper" v-if="!quList.length">
			<p @click="$router.push({ name: 'Edit', params: { id: 0 } })">
				+ 新建问卷
			</p>
		</div>
		<div class="list-wrapper" v-else>
			<ul>
				<li></li>
				<li>问卷标题</li>
				<li>截止时间</li>
				<li>问卷状态</li>
				<li>
					操作
					<p
						@click="
							$router.push({ name: 'Edit', params: { id: 0 } })
						"
					>
						+ 新建问卷
					</p>
				</li>
			</ul>
			<ul v-for="item in quList">
				<li @click="checkItem(item)">
					<i :class="{ checked: item.checked }"></i>
				</li>
				<li v-text="item.title"></li>
				<li v-text="item.time"></li>
				<li
					v-text="item.stateName"
					:class="{ releasing: item.state === 1 }"
				></li>
				<li>
					<router-link tag="span" :to="`/fill/${item.id}`"
						>查看问卷</router-link
					>
					<span
						v-if="!item.state"
						@click="
							iterator = editItem(item);
							iterator.next();
						"
						>编辑</span
					>
					<router-link tag="span" v-else :to="`/data/${item.id}`"
						>查看数据</router-link
					>
					<span
						@click="
							iterator = deleteItem(item);
							iterator.next();
						"
						>删除</span
					>
				</li>
			</ul>
			<div>
				<p @click="checkAll(isCheckedAll)">
					<i :class="{ checked: isCheckedAll }"></i>
				</p>
				<p>
					全选<span
						@click="
							iterator = deleteCheckedItems();
							iterator.next();
						"
						>删除</span
					>
				</p>
			</div>
		</div>
		<div class="overlay" v-show="isShowPrompt">
			<div class="prompt-box">
				<header>
					<span>提示</span>
					<a href="javascript:;" @click="isShowPrompt = false"
						>&times;</a
					>
				</header>
				<p>{{ promptText }}</p>
				<footer>
					<span
						@click="
							iterator.next();
							isShowPrompt = false;
						"
						>确定</span
					>
					<span @click="isShowPrompt = false">取消</span>
				</footer>
			</div>
		</div>
	</div>
</template>

<script>
import Vue from "vue";
import Store from "../store.js";
import data from "../data.js";
export default {
	name: "List",
	data() {
		return {
			quList: [],
			iterator: {},
			isShowPrompt: false,
			promptText: ""
		};
	},
	created() {
		let curTime = Date.now();
		this.quList = Store.fetch() || data.list;
		this.quList.forEach(item => {
			if (item.state === 1) {
				let itemTime = new Date(item.time.replace(/-/g, ",")) * 1;
				if (itemTime < curTime) {
					item.state = 2;
					item.stateName = "已结束";
				}
			}
		});
	},
	methods: {
		checkItem(item, flag = null) {
			if (typeof item.checked === "undefined") {
				Vue.set(item, "checked", true);
			} else if (flag !== null) {
				item.checked = !flag;
			} else {
				item.checked = !item.checked;
			}
		},
		checkAll(flag) {
			this.quList.forEach(item => {
				this.checkItem(item, flag);
			});
		},
		showPrompt(text) {
			this.promptText = text;
			this.isShowPrompt = true;
		},
		*deleteItem(item) {
			yield this.showPrompt(`确认要删除《${item.title}》？`);
			let index = this.quList.indexOf(item);
			yield this.quList.splice(index, 1);
		},
		*deleteCheckedItems() {
			let checkedList = this.quList.filter(item => item.checked);
			if (!checkedList.length) {
				return;
			}
			yield this.showPrompt("确认要删除所选问卷？");
			yield (this.quList = this.quList.filter(item => !item.checked));
		},
		*editItem(item) {
			yield this.showPrompt(`确认要编辑《${item.title}》？`);
			yield this.$router.push({ name: "Edit", params: { id: item.id } });
		}
	},
	computed: {
		isCheckedAll() {
			return this.quList.every(item => item.checked);
		}
	},
	watch: {
		quList: {
			handler(list) {
				list.forEach((item, index) => (item.id = index + 1));
				Store.save(list);
			},
			deep: true
		}
	}
};
</script>

<style scoped lang="scss">
@import "../style/public.scss";
.container {
	width: 1200px;
	margin: 72px auto;
	color: #555;
}
.add-wrapper {
	height: 240px;
	@include flex-center;
	@include wrap-background;
	p {
		padding: 12px 36px;
		@include add-btn;
	}
}
.list-wrapper {
	overflow: hidden;
	@include wrap-background;
	@include check-icon;
	ul {
		display: flex;
		height: 72px;
		line-height: 72px;
		@include nomal-btn;
		&:nth-of-type(1) {
			background-color: #f2f2f2;
			p {
				display: inline-block;
				width: 120px;
				height: 36px;
				margin-left: 132px;
				line-height: 36px;
				text-align: center;
				@include add-btn;
			}
		}
		&:not(:first-child) {
			border-bottom: 1px solid #eee;
			&:hover {
				border: 1px solid rgba(48, 166, 245, 1);
				box-shadow: 0 0 10px rgba(48, 166, 245, 1);
			}
		}
		li {
			&:nth-of-type(1) {
				text-align: center;
				flex: 1;
			}
			&:nth-of-type(2) {
				flex: 5;
			}
			&:nth-of-type(3) {
				flex: 3;
			}
			&:nth-of-type(4) {
				flex: 2;
			}
			&:nth-of-type(5) {
				flex: 4;
			}
		}
		.releasing {
			color: $light-green;
		}
	}
	div {
		display: flex;
		height: 84px;
		line-height: 72px;
		@include nomal-btn();
		p {
			&:nth-of-type(1) {
				text-align: center;
				flex: 1;
			}
			&:nth-of-type(2) {
				flex: 14;
				span {
					margin-left: 24px;
				}
			}
		}
	}
}
</style>
