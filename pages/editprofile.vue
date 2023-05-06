<template>
  <v-sheet width="300" class="mx-auto mt-7">
    <v-form fast-fail ref="form" @submit.prevent="submitForm">
      <v-text-field v-model="email" label="email" :rules="emailRules" required disabled
        @input="resetEmailError"></v-text-field>
      <v-text-field v-model="nickname" label="nickname" :rules="nicknameRules" required></v-text-field>
      <v-btn type="submit" block class="mt-5">닉네임 수정</v-btn>
      <v-btn block class="mt-5" to="/editpassword">비밀번호 변경</v-btn>
    </v-form>
  </v-sheet>
</template>
  
<script>
export default {
  middleware: 'auth',
  data: () => ({
    email: '',
    emailError: '',
    nickname: '',
    password: '',
    nicknameRules: [
      value => !!value || '닉네임을 입력해주세요.',
      value => (value && value.length >= 2 && value.length <= 20) || '닉네임은 2자 이상 20자 이하로 구성되어야 합니다.',
    ],
  }),
  mounted() {
    this.getUserInfo();
  },
  computed: {
    emailRules() {
      return [
        value => !!value || '이메일을 입력해주세요.',
        value => /^.+@.+\..{2,}$/.test(value) || '이메일이 올바른 형식이 아닙니다.',
        value => !this.emailError || this.emailError,
      ]
    }
  },
  methods: {
    resetEmailError() {
      if (this.emailError) {
        this.emailError = '';
        this.$refs.form.validate();
      }
    },
    getUserInfo() {
      this.email = this.$auth.user.email;
      this.nickname = this.$auth.user.nickname;
    },
    async submitForm() {
      if (this.$refs.form.validate()) {
        try {
          const response = await this.$axios.$post('/users/profile', {
            nickname: this.nickname,
          });
          if (response.ok === false) {
            this.emailError = response.error;
            this.$refs.form.validate();
          }
          else {
            this.$root.$emit('showSnackbar', '정보 수정이 성공적으로 완료되었습니다.', 'blue', 5000);
            await this.$auth.fetchUser();
            this.$router.push('/');
          }
        } catch (error) {
          console.error('Signup failed:', error);
          if (error.response) {
            this.$root.$emit('showSnackbar', '정보 수정에 실패했습니다. 다시 시도해주세요.', 'red', 5000);
          } else {
            this.$root.$emit('showSnackbar', '네트워크 에러가 발생했습니다. 다시 시도해주세요.', 'red', 5000);
          }
          this.$router.push('/');
        }
      }
    },
  },
}
</script>