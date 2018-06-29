<template>
	<div class="message-con">
			<Dropdown trigger="click">
				<a href="javascript:void(0)">
					<Badge :count="value">
						<Icon type="ios-bell" :size="22"></Icon>
					</Badge>
				</a>
				<DropdownMenu slot="list">
					<DropdownItem v-for="(item, index) in backlogArr" :key="item.type"> 
						<Row>
							<Col span="8">
								<span>{{ item.type }}</span>
							</Col>
							<Col span="8">
								<Badge :count="item.num"></Badge>
							</Col>
							<Col span="8">
								<Button type="info" size="small" @click="skip(item.router)">立即处理</Button>
							</Col>
						</Row>
					</DropdownItem>
				</DropdownMenu>
			</Dropdown>
	</div>
</template>

<script>

import { backlog } from "@/service/getData";

export default {
	name: "messageTip",
	data() {
		return {
			value: 0,
			backlogArr: [],
			backlogData: {}
		}
	},
  methods: {
		skip(adr) {
			this.$router.push({ path: adr, query: { from: 'bell' } });
		}
	},
	created() {
		backlog()
		.then(res => {
			if(!res.code) {
				this.backlogData = res.data;
				for (let key in res.data) {
					this.value += res.data[key];
					let obj = {};
					if(key==='applicationNum') {
						this.backlogArr.push({ type: '实名审核', num: res.data[key], router: '/member/memberaudit' });
					}else if(key==='withdrawRecordNum') {
						this.backlogArr.push({ type: '提币审核', num: res.data[key], router: '/finance/userwithdrawals' });
					}else if(key==='businessAuthNum') {
						this.backlogArr.push({ type: '商家审核', num: res.data[key], router: '/otc/businessaudit' });
					}else if(key==='appealNum') {
						this.backlogArr.push({ type: '订单申诉', num: res.data[key], router: '/otc/appealManage' });
					}else if(key==='businessCancelNum') {
						this.backlogArr.push({ type: '退保审核', num: res.data[key], router: '/otc/cancelapply' });
					}
				} 
			}else this.$Message.error(res.message);
		})
		.catch(err => console.log(err))
	}
};
</script>

<style lang="less" scoped>
	.ivu-dropdown-rel{
		a{
			color: rgb(73,80,96);
		}
	}
	.ivu-dropdown-item{
		.ivu-row{
			width: 190px;
		}
	}
</style>

