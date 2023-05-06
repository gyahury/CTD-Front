<template>
  <div>
    <div class="content_wrapper">
      <div v-if="todos.length === 0">
        <p v-if="!this.$auth.loggedIn">로그인이 필요합니다.</p>
        <p v-else>{{ this.$auth.user.nickname }}님 ! <br />하단의 To Do를 입력해보세요.</p>
      </div>
      <div v-else>
        <v-card v-if="todos.length > 0">
          <div v-for="(todoObj, i) in todos" :key="`${i}-${todoObj.content}`">
            <v-divider v-if="i !== 0" :key="`${i}-divider`" class="custom-divider"></v-divider>
            <v-list-item>
              <v-list-item-action>
                <v-checkbox class="mt-5" v-model="todoObj.status" :color="todoObj.status == 1 && 'grey' || 'primary'"
                  @change="updateTodo(todoObj)">
                  <template v-slot:label>
                    <div :style="{ textDecoration: todoObj.status == 1 ? 'line-through' : 'none' }" class="ms-4 txt_area"
                      v-text="todoObj.content">
                    </div>
                  </template>
                </v-checkbox>
              </v-list-item-action>
              <v-spacer></v-spacer>
              <v-scroll-x-transition>
                <v-icon v-if="todoObj.status == 1" color="success" style="width:43px">
                  mdi-check
                </v-icon>
              </v-scroll-x-transition>
              <v-list-item-icon style="margin:auto;"><v-icon>{{ icon(todoObj.feel) }}</v-icon></v-list-item-icon>
            </v-list-item>
          </div>
        </v-card>
      </div>
    </div>
    <v-fab-transition>
      <v-btn v-if="!this.$auth.loggedIn" color="red" to="/login" fab dark small fixed
        :style="{ right: 'calc(50% - 200px)', bottom: '150px' }" class="hide-on-mobile">
        <v-icon>mdi-account-key</v-icon>
      </v-btn>
      <v-btn v-if="this.$auth.loggedIn" color="green" fab dark small fixed
        :style="{ right: 'calc(50% - 200px)', bottom: '150px' }" to="/me" class="hide-on-mobile">
        <v-icon>mdi-account</v-icon>
      </v-btn>
    </v-fab-transition>
    <div class="input_wrapper">
      <TodoInput ref="TodoInput" v-on:todoAdded="addTodo" />
    </div>
  </div>
</template>

<script>
import TodoInput from '~/components/TodoInput.vue';
export default {
  async asyncData({ app }) {
    try {
      if (app.$auth.loggedIn) {
        const response = await app.$axios.$get('/todos', {
          headers: {
            Authorization: `${app.$auth.getToken('local')}`,
          },
        });
        return { todos: response.data.todos, userId: response.data.userId };
      }
      return { todos: [] };
    } catch (error) {
      console.error('API 호출 중 오류 발생:', error);
      return { todos: [] };
    }
  },

  components: {
    TodoInput,
  },

  data: () => ({
    todos: [],
  }),

  methods: {
    async updateTodo(todoObj) {
      try {
        const statusValue = todoObj.status ? 1 : 0; //status 숫자로 치환   
        const todoId = parseInt(todoObj.id); 
        const response = await this.$axios.$patch(`/todos/${todoId}`, {
          status: statusValue,
        }, {
          headers: {
            Authorization: `${this.$auth.getToken('local')}`,
          },
        });
        if (response.ok === true) {
          this.$root.$emit('showSnackbar', 'todo 업데이트 되었습니다.', 'green', 5000);
        } else {
          this.$root.$emit('showSnackbar', 'todo 업데이트에 실패했습니다.', 'red', 5000);
        }
      } catch (error) {
        console.error('API 호출 중 오류 발생:', error);
        this.$root.$emit('showSnackbar', 'todo 업데이트에 실패했습니다.', 'red', 5000);
      }
    },
    icon(feel) {
      const iconNames = [
        'mdi-emoticon-outline',
        'mdi-emoticon-cool-outline',
        'mdi-emoticon-dead-outline',
        'mdi-emoticon-excited-outline',
        'mdi-emoticon-happy-outline',
        'mdi-emoticon-neutral-outline',
        'mdi-emoticon-sad-outline',
        'mdi-emoticon-tongue-outline',
      ];
      return iconNames[feel];
    },
    addTodo(todo) {
      this.todos.unshift(todo);
    },
  },
};
</script>

<style scoped lang="scss">
.content_wrapper {
  padding-bottom: 10vh;
  padding-top: 1vh;
  padding-left: 5vh;
  padding-right: 5vh;
}

.input_wrapper {
  width: 450px;
  position: fixed;
  bottom: 0.6vh;
  transform: translateY(-25%);
  z-index: 1;
}

p {
  margin-top: 10px;
  text-align: center;
  font-weight: bold;
  color: rgba(22, 22, 22, 0.6);
}

.v-list-item {
  word-break: break-all;
}

.txt_area {
  width: 234px
}


@media (max-width: 768px) {
  .content_wrapper {
    padding-left: 2vh;
    padding-right: 2vh;
    width: 100%;
    min-height: 80vh;
    overflow-y: auto;
  }

  .input_wrapper {
    text-align: center;
    max-width: 100%;
  }

  .txt_area {
    max-width: 100%;
  }

  .hide-on-mobile {
    display: none;
  }
}
</style>