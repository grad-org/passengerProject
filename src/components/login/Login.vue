<template>
	<div>
		<!-- <button @click="goHome"><logo id="logo" height="48px" ></logo></button> -->
		<span @click="goHome"><logo id="logo" height="48px" ></logo></span>
		<div id="container">
			<mu-col width="100%" tablet="100" desktop="100">
				<label id="title">登录</label><br>
				<label id="info">使用您的网约车账号</label>
			</mu-col>
			<mu-col>
				<mu-text-field label="输入手机号码" labelFloat v-model="value_username" :errorText="errorText_username" :fullWidth="fullWidth" @focus="onFocusUsername"/>
				<!-- 您所输入的是：{{value_username}} -->
			</mu-col>
			<mu-col style="margin: 24px 0">
				<!-- <mu-raised-button label="创建账号" /> -->
				<label id="btn_left" @click="goRegister">创建账号</label>
				<mu-raised-button label="下一步" @click="loginNext" />
			</mu-col>
			<mu-dialog :open="dialog" title="错误提示">
				{{errorTips}}
				<mu-flat-button label="确定" slot="actions" primary @click="closeDialog"/>
			</mu-dialog>
		</div>
	</div>
</template>

<script>
	
	import Logo from '../../assets/Logo'
	import checkFormat from './js/CheckFormat.js';
	
	export default {
		name: 'Login',
		components: {
			Logo
		},
		data () {
			return {
				fullWidth: true,
				value_username: '',
				errorText_username: '',

				dialog: false,
				errorTips: ''
			}
		},
		created () {
			// console.log('读取Login的token：' + this.$store.state.token)
		},
		methods: {
			goHome () {
				this.$router.push({name: 'Home'});
			},
			goRegister () {
				this.$router.push({name: 'Register'});
			},
			onFocusUsername () {
				this.errorText_username = ''
			},
			loginNext () {
				let _this = this;
				_this.errorText_username = checkFormat.checkUsername(_this.value_username);
				if (_this.errorText_username == '') {
					// 验证用户是否存在前，需要将token删除，否则会出现500错误！
					_this.$store.dispatch('logout')
					_this.$axios.get('/api/auth/verify?username=' + _this.value_username)
					.then((response) => {
						console.log(response)
						if (response.status == 200) {
							_this.$router.push({path: '/login/2', name: 'LoginNext', params: { username: _this.value_username }});
						}
					})
					.catch((error) => {
						console.log(error)
						if (error.status == 400) {
							_this.errorTips = '用户不存在！';
							_this.dialog = true;
							console.log("用户不存在");
						}
						if (error.status == 500) {
							_this.errorTips = '服务器内部错误！';
							_this.dialog = true;
							console.log('服务器内部错误（' + error.status +'）');
						}
					})

				} else {
					this.errorText_username = checkFormat.checkUsername(this.value_username);
				}
				
			},
			closeDialog () {
				this.dialog = false
			}
		}
	}
</script>


<style scoped>
	@import url(./css/login.css);
</style>
