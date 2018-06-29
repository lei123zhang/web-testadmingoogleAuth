<template>
  <div class="googleAuth">
    <Card>
      <p slot="title">
        谷歌验证
      </p>
		<Row>
			<Col span="8" offset="8" class="infoBoard" v-if="!readyBind">
				<img  :src="this.qrcodeUrl" alt="googleAuth">
				<div class="info">
					<p>
						用户名：{{ this.link | filterLink }}
					</p>
					<p>
						密钥：{{this.secretKey}}
					</p>
				</div>
			</Col>
		</Row>
		<br><br>
		<Row v-if="!readyBind">
			<Col span="6" offset="9">
				<Input v-model="code" placeholder="请输入谷歌验证码..."></Input>
				<br><br>
				<Button @click="bindAuth" type="success" long>绑 定</Button>
			</Col>
		</Row>
		<Row v-if="!!readyBind">
			<Col span="6" offset="9">
			<h3>您已经绑定google auth！</h3>
				<Input v-model="code" placeholder="请输入谷歌验证码..."></Input>
				<br><br>
				<Input v-model="password" placeholder="请输入登录密码..."></Input>
				<br><br>
				<Button @click="unbindGoogleFn" type="error" long>解除绑定</Button>
			</Col>
		</Row>
    </Card>
    </div>
</template>


<script>

import QRCode from 'qrcode'
import { sendGoogle, bindGoogle, unbindGoogle, googleState } from '@/service/getData';
import { error } from 'util';

export default {
	name: "googleAuth",
	data() {
		return {
			qrcodeUrl: '',
			secretKey: '',
			link: '',
			code: '',
			loginCode:　'',
			password: '',
			readyBind: 0 //0 F 1 T
		}
	},
	methods: {
		bindAuth() {
			bindGoogle({codes: this.code, secret: this.secretKey})
			.then(res => {
				this.$Message.info(res.message);
				this.initData();
				this.code =  '';
			})
			.catch(err => {
				console.error(err);
			}) 
		},
		unbindGoogleFn() {
			unbindGoogle({codes: this.code,	password: this.password	}) 
			.then(res => {
				this.$Message.info(res.message);
				this.initData();
				this.code =  '';
			})
			.catch(err => {
				console.error(err);
			})
		},
		initData() {
			googleState()
			.then(res => {
				this.readyBind	=  res.data;
			})

			sendGoogle() 
			.then(res => {
				if(!res.code) {
					this.secretKey = res.data.secret;
					this.link = res.data.link;
					QRCode.toDataURL(res.data.link)
					.then( url => {
						this.qrcodeUrl = url;
					})
				}else this.$Message.error(res.message);
			})
			.catch( err => {
				console.error(err);
			})
		}
	},
	created() {
		this.initData();
	},
	filters: {
		filterLink(val) {
			if(!!val) {
				let userName = val.split('otpauth://totp/')[1].split('?secret')[0];
				return userName;
			}else return val;
		}
	}
}
</script>

<style lang="less" scoped>
	.googleAuth{
		h3{
			text-align: center;
    	margin-bottom: 20px;
		}
		.infoBoard{
			.info{
				display: inline-block;
			}
		}
	}
</style>

