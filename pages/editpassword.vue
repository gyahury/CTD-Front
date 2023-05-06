<template>
    <v-sheet width="300" class="mx-auto mt-7">
        <v-form fast-fail ref="form" @submit.prevent="submitForm">
            <v-text-field v-model="password" label="password" type="password" :rules="passwordRules"
                required></v-text-field>
            <v-text-field v-model="confirmPassword" label="confirm password" type="password" :rules="confirmPasswordRules"
                required></v-text-field>
            <v-btn type="submit" block class="mt-5">비밀번호 변경</v-btn>
        </v-form>
    </v-sheet>
</template>

<script>
export default {
    middleware: 'auth',

    data: () => ({
        password: '',
        passwordRules: [
            value => !!value || '비밀번호를 입력해주세요.',
            value => (value && value.length >= 8 && value.length <= 15) || '비밀번호는 8자 이상 15자 이하로 구성되어야 합니다.',
            value => /[A-Z]/.test(value) || '비밀번호는 대문자가 포함되어야 합니다.',
            value => /[!@#$%^&*(),.?":{}|<>]/.test(value) || '비밀번호는 특수문자가 포함되어야 합니다.'
        ],
        confirmPassword: '',
    }),
    computed: {
        confirmPasswordRules() {
            return [
                value => !!value || '비밀번호를 입력해주세요.',
                value => value === this.password || '비밀번호가 일치하지 않습니다.'
            ];
        },
    },
    methods: {
        async submitForm() {
            if (this.$refs.form.validate()) {
                try {
                    const response = await this.$axios.$post('/users/profile', {
                        password: this.password,
                    });
                    if (response.ok === false) {
                        this.$refs.form.validate();
                        this.$root.$emit('showSnackbar', '비밀번호 변경이 실패했습니다. 다시 시도해주세요.', 'red', 5000);
                    }
                    else {
                        this.$root.$emit('showSnackbar', '비밀번호 변경이 성공적으로 완료되었습니다.', 'blue', 5000);
                        await this.$auth.fetchUser();
                        this.$router.push('/');
                    }
                } catch (error) {
                    console.error('Signup failed:', error);
                    if (error.response) {
                        this.$root.$emit('showSnackbar', '비밀번호 변경이 실패했습니다. 다시 시도해주세요.', 'red', 5000);
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