<template>
    <v-card class="fill-height">
        <v-card-title>
            <span class="text-h5">AI Advice for you</span>
        </v-card-title>
        <v-card-text class="mt-5">
            <div v-if="coachingContent" v-html="formattedcoachingContent"></div>
            <template v-else>
                시간이 10초 정도 소요됩니다. 잠시만 기다려주세요.
            </template>
        </v-card-text>
    </v-card>
</template>

<script>

export default {
    middleware: 'auth',
    data() {
        return {
            coachingContent: '',
        };
    },
    computed: {
        formattedcoachingContent() {
            return this.coachingContent.replace(/\n/g, '<br>');
        },
    },
    async mounted() {
        try {
            const response = await this.$axios.$post('/ai/claude');
            if(response.ok){
                this.coachingContent = response.data.claude;
            }else{
                this.$router.push('/');
                this.$root.$emit('showSnackbar', '목록이 없습니다.', 'red', 5000);
            }
                
            
            
        } catch (error) {
            console.error(error);
            this.$root.$emit('showSnackbar', '네트워크 에러가 발생했습니다. 다시 시도해주세요.', 'red', 5000);
        }
    },
}
</script>