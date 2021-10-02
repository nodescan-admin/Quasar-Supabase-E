<!-- Translations from original Russian
'Введите адрес электронной почты' 'Please enter your email address'
'Введите адрес электронной почты в качестве логина для входа в программу' 'Enter your email address as a login to enter the program'
'Укажите пароль для регистрации в программе' Enter the password to register in the program
'Введите пароль для входа в программу'  Enter the password to enter the program
'Пароль повторно' 'Password again'
'Пароль' 'Password'
'Введите пароль повторно' 'Please re-enter your password'
'Пароли не совпадают' 'Password mismatch'
'Войти с помощью' 'Login with'
'Создать новый аккаунт' 'Create a new account'
'Я не помню пароль' 'I do not remember the password'
'Повторите ввод пароля для регистрации в программе' 'Re-enter the password to enter the program'
'У меня есть учетная запись' I already have an account

Removed from password.
Change to conditional - Enable rule only if MagicLinkEnabled is false
   :rules='[val => !!val || "enter password"]'
-->

<template lang='pug'>
q-page(padding)
  h5.text-center.text-primary.q-mt-lg {{ getAuthTitle }}
  q-form.authentication.q-px-md.q-gutter-y-sm(ref='emailAuthenticationForm' @submit='onSubmit')
    q-input(
      v-model='email'
      outlined
      color='primary'
      lazy-rules
      label='Email'
      type='email'
      :rules='[val => !!val || "Please enter your email address"]'
      )
        template(v-slot:prepend)
          q-icon(name='far fa-envelope')
        q-tooltip Enter your email address as a login to the service
    q-input(
      v-model='password'
      lazy-rules
      outlined
      color='primary'
      label='Password'
      :type='isPwd ? "password" : "text"'
      @keyup.enter='onSubmit'
    )
      template(v-slot:prepend)
        q-icon(name='fas fa-lock')
      template(v-slot:append)
        q-icon.cursor-pointer(
          :name='isPwd ? "visibility_off" : "visibility"'
          @click='isPwd = !isPwd'
        )
      q-tooltip {{ isRegistration ? 'Enter a password to register for the service' : 'Enter a password to connect to the service'}}
    q-input(
      v-if='isRegistration'
      lazy-rules
      outlined
      color="primary"
      label="Reenter Password"
      v-model="passwordMatch"
      :rules='[ val => !!val || "Please re-enter your password", val => val === password || "Password mismatch" ]'
      :type="isPwd ? 'password' : 'text'"
      @keyup.enter='onSubmit'
    )
      template(v-slot:prepend)
        q-icon(name='fas fa-lock')
      template(v-slot:append)
        q-icon.cursor-pointer(
          :name='isPwd ? "visibility_off" : "visibility"'
          @click='isPwd = !isPwd'
        )
      q-tooltip Re-enter your password to register
    q-btn.full-width.q-mt-sm(
      v-if='magicLinkIsEnabled'
      color='green'
      padding='md xs'
      type='submit'
      :disable='isLoginDisabled'
      :icon='isRegistration ? "fas fa-user-plus" : "fas fa-sign-in-alt"'
      :label='getMagicLinkButtonLabel'
    )
    q-btn.full-width.q-mt-sm(
      v-if='magicLinkSent'
      color='white'
      padding='md xs'
      disable
      :label='getMagicLinkCheckEmailLabel'
    )
    q-btn.full-width.q-mt-sm(
      color='primary'
      padding='md xs'
      type='submit'
      :disable='isLoginDisabled'
      :icon='isRegistration ? "fas fa-user-plus" : "fas fa-sign-in-alt"'
      :label='getAuthButtonLabel'
    )
    q-btn.full-width.q-mt-sm(
      color='secondary'
      padding='md xs'
      label='Login with Google'
      icon='fab fa-google'
      @click='handleLoginProvider("google")'
      )
    q-btn.full-width.q-mt-sm(
      color='dark'
      padding='md xs'
      label='Login with Github'
      icon='fab fa-github'
      @click='handleLoginProvider("github")'
      )
    .q-ml-lg.q-mt-lg.text-weight-medium
      p
        router-link.text-blue(:to='routeAuthentication')
          span(v-if='isRegistration') I already have an account
          span(v-else) Create a new account
      p
        router-link.text-blue(to='forgot-password') I forgot my password
</template>

<script>
import { mapActions } from 'vuex'
import { loadingSpinner } from '../../mixins'
import { useQuasar } from 'quasar'

// const $q = useQuasar()

export default {
  mixins: [loadingSpinner],
  name: 'UserAuth',
  magicLinkSent: false,
  setup () {
    const $q = useQuasar()
    $q.notify('Message')
  },
  computed: {
    // Change to false to remove MagicLink option
    magicLinkIsEnabled () {
      return true
    },
    isLoginDisabled () {
      if (!this.email) {
        return true
      }
      return !this.validateEmail()
    },
    getAuthTitle () {
      // return this.isRegistration ? 'Регистрация в программе' : 'Вход в программу'
      return this.isRegistration ? 'Register' : 'Login'
    },
    getMagicLinkCheckEmailLabel () {
      return 'Check your email for the login link'
    },
    getMagicLinkButtonLabel () {
      return this.isRegistration ? 'Register without password' : 'Log in with Magic link'
    },
    getAuthButtonLabel () {
      // return this.isRegistration ? 'Зарегистрироваться' : 'Войти'
      return this.isRegistration ? 'Register with password' : 'Log in'
    },
    isRegistration () {
      return this.$route.name === 'Register'
    },
    routeAuthentication () {
      return this.isRegistration ? '/auth/login' : '/auth/register'
    }
  },
  data () {
    return {
      email: '', // null
      isPwd: true,
      password: null,
      passwordMatch: null
    }
  },
  methods: {
    ...mapActions({
      showError: 'logs/showError'
    }),
    validateEmail () {
      // eslint-disable-next-line
      return (/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(this.email))
    },
    onSubmit () {
      const { email, password } = this
      this.magicLinkSent = false
      this.$refs.emailAuthenticationForm.validate()
        .then(async success => {
          if (!success) return
          this.isLoading = true
          try {
            if (this.isRegistration) {
              const { error } = await this.$supabase.auth.signUp({ email, password })
              if (error) throw new Error(error)
            } else {
              const { error } = await this.$supabase.auth.signIn({ email, password })
              if (error) throw new Error(error)
            }
            // this.$nextTick(() => this.$router.push({ path: '/' }))
          } catch (err) {
            this.showError(err)
            if (!password) {
              this.magicLinkSent = true
              // alert('Please check your email for the login link')
              this.$q.notify('A magic (login) link was not sent. Please check your email address or register')
            }
          } finally {
            this.isLoading = false
            if (!password) {
              this.magicLinkSent = true
              // alert('Please check your email for the login link')
              this.$q.notify('Please check your email for the login link')
            }
          }
        })
    },
    async handleLoginProvider (provider) {
      this.isLoading = true
      try {
        const { error } = await this.$supabase.auth.signIn({
          provider
        })
        if (error) throw new Error(error)
      } catch (err) {
        this.showError(err)
      } finally {
        this.isLoading = false
      }
    }
  }
}
</script>

<style lang='scss' scoped>
.authentication {
  margin: auto;
  max-width: 25rem;
  width: 100%;
}
</style>
