<template>
    <v-form @submit.prevent>
        <v-container>
            <v-text-field ref="todoInput" v-model="todo" :append-outer-icon="'mdi-plus'" :prepend-icon="icon" filled
                clear-icon="mdi-close-circle" clearable label="To Do" type="text" color="rgb(22,22,22)"
                @click:append-outer="debouncedSendTodo" @click:prepend="changeIcon" @click:clear="clearTodo"
                @keydown.enter.prevent="debouncedSendTodo"></v-text-field>
        </v-container>
    </v-form>
</template>
  
<script>
import { debounce } from "lodash";
export default {
    data: () => ({
        todo: '',
        isDone: 0,
        iconIndex: 0,
        icons: [
            'mdi-emoticon-outline',
            'mdi-emoticon-cool-outline',
            'mdi-emoticon-dead-outline',
            'mdi-emoticon-excited-outline',
            'mdi-emoticon-happy-outline',
            'mdi-emoticon-neutral-outline',
            'mdi-emoticon-sad-outline',
            'mdi-emoticon-tongue-outline',
        ],
    }),

    computed: {
        icon() {
            return this.icons[this.iconIndex]
        },
    },

    methods: {
        debouncedSendTodo: debounce(async function () {
            await this.sendTodo();
        }, 100),

        async sendTodo() {

            // 빈 값인 경우
            if (this.todo.trim() === '') {
                this.$root.$emit('showSnackbar', '내용을 입력해주세요.', 'red', 3000);
                this.$refs.todoInput.focus();
                return;
            }

            try {
                if (this.$root.$auth.loggedIn) {
                    const response = await this.$axios.$post('/todos', {
                        content: this.todo,
                        feel: this.iconIndex,
                    }, {
                        headers: {
                            Authorization: `${this.$root.$auth.getToken('local')}`,
                        },
                    });
                    this.$emit('todoAdded', {
                        id : response.todoId,
                        content: this.todo,
                        feel: this.iconIndex,
                        status: this.isDone,
                    });
                    this.resetIcon();
                    this.clearTodo();
                    this.$refs.todoInput.focus();
                } else {
                    this.$root.$emit('showSnackbar', '로그인하셔야합니다.', 'red', 5000);
                }
            } catch (error) {
                this.$root.$emit('showSnackbar', '서버와의 연결에 실패하였습니다.', 'red', 5000);
            }
        },
        clearTodo() {
            this.resetIcon()
            this.todo = ''
        },
        resetIcon() {
            this.iconIndex = 0
        },
        changeIcon() {
            this.iconIndex === this.icons.length - 1
                ? this.iconIndex = 0
                : this.iconIndex++
        },
    },
}
</script>