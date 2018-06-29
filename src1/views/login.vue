<style lang="less">
@import "./login.less";
</style>

<template>
	<div class="login" @keydown.enter="handle">
		<div class="login-con">
			<Card :bordered="false">
				<p slot="title">
					<Icon type="log-in"></Icon> 欢迎登录
				</p>
				<div class="form-con">
					<Form ref="loginForm" :model="form" :rules="rules">
						<div v-if="googleAuth!==null">
							<FormItem prop="username">
								<Input v-model="form.username" :disabled="btnDisable" placeholder="请输入用户名">
									<span slot="prepend">
										<Icon :size="16" type="person"></Icon>
									</span>
								</Input>
							</FormItem>

							<FormItem prop="password">
								<Input type="password" v-model="form.password" :disabled="btnDisable" placeholder="请输入密码">
									<span slot="prepend">
										<Icon :size="14" type="locked"></Icon>
									</span>
								</Input>
							</FormItem>

						</div>

						<FormItem prop="googleAuth" v-if="googleAuth===null">
							<Input v-model="form.codes" placeholder="请输入谷歌验证码">
								<span slot="prepend">
									<Icon :size="14" type="locked"></Icon>
								</span>
							</Input>
						</FormItem>

						<FormItem style='margin-top:10px'>
							<Button @click="handle" type="primary" long>登录</Button>
						</FormItem>

					</Form>
				</div>
			</Card>
		</div>
	</div>
</template>

<script>
import Cookies from "js-cookie";
import store from "../store";

import { setStore, getStore, removeStore } from "@/config/storage";
import { otherRouter, appRouter } from "@/router/router.js";
import { onlyGoogleLogin, BASICURL, getLoginCode, signIn, getCodeAgain, googleAuthLogin, ifBindGoogleAuth } from "@/service/getData";
export default {
  data() {
    return {
			timer: '',
			btnDisable: false,
			count: 0,
      form: {
        username: '',
				password: '',
				codes: ''
			},
			code: null,
			googleAuth: '',
      messshow: false,
      errormessage: null,
      logimg: `${BASICURL}/admin/captcha?cid=ADMIN_LOGIN`,
      rules: {
        username: [{ required: true, message: "不能为空", trigger: "blur" }],
        password: [{ required: true, message: "不能为空", trigger: "blur" }]
      },
      permissions: {}
    };
  },
  methods: {
		cancelSignIn() {
			Cookies.remove('userPhone');
			window.location.reload();
		},
    handle() {
			if(this.googleAuth===null) {
				console.log(1)
				onlyGoogleLogin({codes: this.form.codes})
					.then(res=> {
						if(!res.code) {
							Cookies.set('user', res.data.admin.username, { expires: 7 });
							Cookies.set('userInfo', res.data.admin, { expires: 7 });
							setStore("leftSidebarList", res.data.permissions);
							this.$router.push({ name: "home_index" });
						}
					})
			}else{
				console.log(2)
				
				googleAuthLogin(this.form)
				.then( res => {
					if(!res.code) {
						this.googleAuth = res.data;
						if(res.data!==null) {
							Cookies.set('user', res.data.admin.username, { expires: 7 });
							Cookies.set('userInfo', res.data.admin, { expires: 7 });
							setStore("leftSidebarList", res.data.permissions);
							this.$router.push({ name: "home_index" });
						}else this.googleAuth===null;
					}else this.$Message.error(res.message)
				})
			}
    }
  },
  beforeRouteLeave(to, from, next) {
		clearInterval(this.timer)
    window.location.reload();
    next();
  },
  created() {
		this.phoneNum = Cookies.get('userPhone');
		clearInterval(this.timer)
		this.logimg = `${BASICURL}admin/captcha?cid=ADMIN_LOGIN`;
	},
	filters: {
		hidePhoneNum(val) {
			return val.split('').fill('*',3,7).join('');
		}
	},
	watch: {
		count(newVal, oldVal) {
			if(newVal>0){
				this.timer = setTimeout(()=> {
					this.count--;
				}, 1000)
			}else{
				clearInterval(this.timer)
			}
		},
		phoneNum(newVal, oldVal) {
			if(!!newVal) this.count = 0;
		}
	}
};
</script>