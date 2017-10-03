<template>
	<div id="login">
		<div class="columns is-half-desktop">
			<div class="column">&nbsp;</div>
			<div class="column">
				<h2 class="title">Login</h2>
				<div class="field">
				  <p class="control has-icons-left has-icons-right">
				    <input class="input" type="email" placeholder="Email" v-model="email">
				    <span class="icon is-small is-left">
				      <i class="fa fa-envelope"></i>
				    </span>
				    <span class="icon is-small is-right">
				      <i class="fa fa-check"></i>
				    </span>
				  </p>
				</div>
				<div class="field">
				  <p class="control has-icons-left">
				    <input class="input" type="password" placeholder="Password" v-model="password">
				    <span class="icon is-small is-left">
				      <i class="fa fa-lock"></i>
				    </span>
				  </p>
				</div>
				<div class="field">
				  <p class="control">
				    <button class="button is-primary" v-on:click="submitLogin">
				      Login
				    </button>
				  </p>
				</div>
			</div>
			<div class="column">&nbsp;</div>
		</div>
	</div>
</template>
<script>
	import axios from 'axios';
	export default {
		name: 'login',
		data: () => ({
			email: '',
			password: ''
		}),
		methods: {
			submitLogin(){
				axios.post('https://localhost:3005/v1/account/login', {
				    email: this.email,
				    password: this.password
				  })
				  .then(function (response) {
				    console.log(response);
				  })
				  .catch(function (error) {
				    console.log(error);
				  });
			}
		}
	}
</script>
<style>
	
</style>