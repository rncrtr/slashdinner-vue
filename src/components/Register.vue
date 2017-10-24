<template>
	<div id="login">
		<div class="columns">
			<div class="column">&nbsp;</div>
			<div class="column">
				<h2 class="title">Register</h2>
				<div class="field">
				  <p class="control has-icons-left has-icons-right">
				    <input class="input" type="email" name="email" placeholder="Email" v-model="email" v-validate="'required|email'">
            <span v-show="errors.has('email') && showErrors" class="help is-danger"><i class="fa fa-warning"></i>&nbsp;{{ errors.first('email') }}</span>
				    <span class="icon is-small is-left">
				      <i class="fa fa-envelope"></i>
				    </span>
				    <span class="icon is-small is-right">
				      <i class="fa fa-check"></i>
				    </span>
				  </p>
				</div>
				<br />
				<div class="field">
				  <p class="control has-icons-left">
				    <input class="input" type="password" name="password" placeholder="Password" v-model="password" v-validate="'required|confirmed:passconfirm'">
				    <span v-show="errors.has('password') && showErrors" class="help is-danger"><i class="fa fa-warning"></i>&nbsp;{{ errors.first('password') }}</span>
				    <span class="icon is-small is-left">
				      <i class="fa fa-lock"></i>
				    </span>
				  </p>
				</div>
				<div class="field">
				  <p class="control has-icons-left">
				    <input class="input" type="password" name="passconfirm" placeholder="Confirm Password" v-model="passconfirm" v-validate="'required'">
				    <span v-show="errors.has('passconfirm') && showErrors" class="help is-danger"><i class="fa fa-warning"></i>&nbsp;{{ errors.first('passconfirm') }}</span>
				    <span class="icon is-small is-left">
				      <i class="fa fa-lock"></i>
				    </span>
				  </p>
				</div>
				<div class="field">
				  <p class="control is-pulled-right">
				    <button class="button is-info" v-on:click="validateBeforeSubmit">
				      Register
				    </button>
				  </p>
				</div>
			</div>
			<div class="column">&nbsp;</div>
		</div>
	</div>
</template>
<script>
	import Vue from 'vue'
	import VeeValidate from 'vee-validate' 
	import axios from 'axios'

	Vue.use(VeeValidate)

	export default {
		name: 'register',
		data () {
			return {
				email: '',
				password: '',
				passconfirm: '',
				showErrors: ''
			}
		},
		methods: {
			submitRegistration () {
				axios.post('/v1/account/register',{
					email: this.email,
					password: this.password
				})
			  .then(response => {
			  	if(response.status==200){
			    	this.$router.push('/')
			    }
			  })
			  .catch(error => {
			    console.log(error);
			  })
			},
			validateBeforeSubmit() {
				this.showErrors = true;
	      this.$validator.validateAll().then((result) => {
	        if (result) {
	          this.submitRegistration()
	          return;
	        }
	      });
    	}
		}
	}
</script>
<style>
	
</style>