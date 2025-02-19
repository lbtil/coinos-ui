<template lang='pug'>
  v-card
    v-card-text
      v-progress-linear(v-if='loading' indeterminate='')
      v-form.mt-4(v-model='validForm' v-else='' @submit.prevent="register()")
        h2.mb-4.white--text Register an Account
        v-text-field.validate(label='Username' v-model='form.username' dark='' autocapitalize='none' ref='username' autocomplete='username' :rules='rules.username' append-icon='$account')
        v-text-field.validate(label='Email' type='email' v-model='form.email' dark='' autocapitalize='none' ref='email' autocomplete='email'  append-icon='$email' :rules='rules.email')
        v-text-field(label='Phone (optional)' @change='validatePhone()' v-model='form.phone' dark='' autocapitalize='none' ref='phone' append-icon='$cellphone' autocomplete='phone' :rules='rules.NAPhone' :class='phoneValidated ? "validated" : "unvalidated"')
        v-text-field.validate(label='Password' v-model='form.password' type='password' ref='password' autocomplete='current-password' :rules='rules.password' append-icon='$lock')
        v-btn-toggle.d-flex.flex-wrap.mx-auto(tile='' color='primary accent-3' group='')
          v-btn.wide.mr-2.flex-grow-1(:disabled='!validForm' type='submit')
            v-icon(left='' color='green') $forward
            span Register
          v-btn.mb-1.mb-sm-0.wide.flex-grow-1(@click="$go('/')")
            v-icon(left='' color='red') $cancel
            span Nevermind
</template>

<script>
import { call, get, sync } from 'vuex-pathify';
const IS_PRODUCTION = !['development', 'test'].includes(process.env.NODE_ENV);
import config from '@/config'

export default {
  props: {
    username: { type: String, default: '' },
    password: { type: String, default: '' },
  },
  data() {
    return {
      validForm: false,
      submitted: false,
      form: {
        username: this.username,
        email: this.email,
        phone: this.phone,
        password: this.password,
        response: '',
        message: ''
      },
      phoneIcon: '$cancel',
      phoneValidated: false,
      rules: {}
    };
  },

  computed: {
    error: sync('error'),
    loading: get('loading'),
  },

  methods: {
    createUser: call('createUser'),
    login: call('login'),
    async register() {
      let user = await this.createUser(this.form);
      user.password = this.form.password;
      console.log("User created...")
      await this.login(user);
      console.log("User logged in...")
      this.$go('/home')
    },
    validatePhone () {
      this.phoneValidated = false
      this.phoneIcon = 'mdi-close'
      var digits = this.form.phone.replace(/\D/g, '')
      console.log(digits.length + ' digits entered: ' + this.form.phone + ' -> ' + digits)
      var formattedPhone
      if (digits.length === 11 && digits.substring(0,1) === '1') {
        formattedPhone = '+1 (' + digits.substring(1,4) + ') ' + digits.substring(4, 7) + '-' + digits.substring(7,11)
      } else if (digits.length === 10) {
        formattedPhone = '+1 (' +  digits.substring(0,3) + ') ' + digits.substring(3, 6) + '-' + digits.substring(6,10)
        this.$set(this.form, 'phone', formattedPhone)
        this.phoneValidated = true
        this.phoneIcon = 'mdi-check'
        console.log('international format: ' + formattedPhone)
      } else if (digits.substring(0,1) === '+') {
        // International number
        formattedPhone = this.form.phone
        console.log('international format entered: ' + formattedPhone)
      }

      if (formattedPhone) {
        console.log('re-formatted: ' + formattedPhone)
        this.$set(this.form, 'phone', formattedPhone)
        this.phoneValidated = true
        this.phoneIcon = '$check'
        this.$set(this.form, 'formattedPhone', formattedPhone)
      } else {
        this.phoneValidated = false
        this.phoneIcon = '$cancel'
      }
    }
  },

  mounted() {

    this.rules = config.rules || {}

    if (!this.username) this.$refs.username.focus();
    else if (!this.password) this.$refs.password.focus();
  },
};
</script>
<style>
.v-text-field.validated .v-input__icon--append .v-icon { 
    color: green;
}
.v-text-field.unvalidated .v-input__icon--append .v-icon { 
    color: red;
}

.v-text-field.validate .v-input__icon--append .v-icon { 
    color: green;
}
.v-text-field.validate.error--text .v-input__icon--append .v-icon { 
    color: red;
}
</style>
