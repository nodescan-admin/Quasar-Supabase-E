<!-- Translations from original Russian
'Change password' 'Новый пароль'
'Enter a new password' 'Введите новый пароль'
'Enter password again 'Новый пароль повторно'
'Re-enter the new password' 'Повторите ввод нового пароля'
'Please re-enter the new password' Введите новый пароль повторно"
'Change password' 'Сменить пароль'
'Восстановление пароля' 'Password recovery'
-->

<template lang='pug'>

q-page(padding)
  h5.text-center.text-primary.q-mt-lg Password recovery
  q-form.authentication.q-px-md.q-gutter-y-sm(ref='changePasswordForm' @submit='onSubmit')
    q-input(
      v-model='password'
      lazy-rules
      outlined
      color='primary'
      label='Change password'
      :rules='[val => !!val || "Enter a new password"]'
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
      q-tooltip Enter a new password
    q-input(
      lazy-rules
      outlined
      color="primary"
      label="Enter password again"
      v-model="passwordMatch"
      :rules='[ val => !!val || "Please re-enter the new password", val => val === password || "Пароли не совпадают" ]'
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
      q-tooltip Re-enter the new password
    q-btn.full-width.q-mt-sm(
      color='primary'
      padding='md xs'
      type='submit'
      icon='fas fa-key'
      label='Change password'
      )
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { loadingSpinner } from '../../mixins'
export default {
  name: 'ChangePassword',
  mixins: [loadingSpinner],
  data () {
    return {
      isPwd: true,
      password: null,
      passwordMatch: null
    }
  },
  computed: {
    ...mapGetters({
      accessToken: 'auth/accessToken'
    })
  },
  methods: {
    ...mapActions({
      showError: 'logs/showError',
      showInfo: 'logs/showInfor'
    }),
    onSubmit () {
      const { password } = this
      this.$refs.changePasswordForm.validate()
        .then(async success => {
          if (!success) return
          this.isLoading = true
          try {
            const { error } = await this.$supabase.auth.api.updateUser(this.accessToken, { password })
            if (error) throw new Error(error)
          } catch (err) {
            this.showError(err)
          } finally {
            this.isLoading = false
          }
        })
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
