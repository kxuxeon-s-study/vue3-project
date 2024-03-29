<template>
  <div v-if="loading">
    Loading...
  </div>
  <form
      v-else
      @submit.prevent="onSave"
  >
    <div class="row">
      <div class="col-6">
        <Input
            label="Subject"
            v-model="todo.subject"
            :error="subjectError"
        />
      </div>
      <div class="col-6">
        <div v-if="editing" class="form-group">
          <label>Status</label>
          <div>
            <button
                class="btn"
                type="button"
                :class="todo.completed ? 'btn-success' : 'btn-danger'"
                @click="toggleTodoStatus"
            >
              {{ todo.completed ? 'Completed' : 'InComplete' }}
            </button>
          </div>
        </div>
      </div>
      <div class="col-12">
        <div class="form-group">
          <label>Body</label>
          <textarea
              v-model="todo.body"
              class="form-control"
              cols="30"
              rows="10"
          ></textarea>
        </div>
      </div>
    </div>
    <button
        type="submit"
        class="btn btn-primary"
        :disabled="!todoUpdated"
    >
      {{ editing ? 'Update' : 'Create' }}
    </button>
    <button
        class="btn btn-outline-dark ms-2"
        @click="moveTodoListPage"
    >
      Cancel
    </button>
  </form>
</template>
<script>
import {useRoute, useRouter} from 'vue-router';
import axios from '@/axios';
import {ref, computed, onUpdated} from "vue";
import _ from 'lodash';
import {useToast} from "@/composables/toast";
import Input from '@/components/Input';

export default {
  components: {
    Input,
  },
  props: {
    editing: {
      type: Boolean,
      default: false
    }
  },
  setup(props) {
    const route = useRoute();
    const router = useRouter();
    const todo = ref({
      subject: '',
      completed: false,
      body: ''
    });
    onUpdated(() => {
      console.log(todo.value.subject);
    })
    const subjectError = ref('');
    const originalTodo = ref(null);
    const loading = ref(false);
    const todoId = route.params.id;
    const {
      toastMessage,
      toastAlertType,
      showToast,
      triggerToast
    } = useToast();

    const getTodo = async () => {
      loading.value = true;
      try {
        const res = await axios.get(`todos/${todoId}`);
        todo.value = {...res.data};
        originalTodo.value = {...res.data};
        loading.value = false;
      } catch (err) {
        loading.value = false
        console.log(err)
        triggerToast('Something went wrong..', 'danger');
      }

    };

    const todoUpdated = computed(() => {
      return !_.isEqual(todo.value, originalTodo.value);
    })


    const toggleTodoStatus = () => {
      todo.value.completed = !todo.value.completed
    }

    const moveTodoListPage = () => {
      router.push({
        name: 'Todos'
      })
    };

    const onSave = async () => {
      subjectError.value = '';
      if (!todo.value.subject) {
        subjectError.value = 'Subject is required'
        return;
      }
      try {
        let res;
        const data = {
          subject: todo.value.subject,
          completed: todo.value.completed,
          body: todo.value.body
        };
        let message = 'Saved !';
        if (props.editing) {
          res = await axios.put(`todos/${todoId}`, data);
          originalTodo.value = {...res.data}
        } else {
          await axios.post('todos', data);
          message = 'Created'
          todo.value.subject = '';
          todo.value.body = '';
        }

        triggerToast('Successfully ' + message);

        if (!props.editing) {
          router.push({
            name: 'Todos'
          });
        }
      } catch (err) {
        triggerToast('Something went wrong..', 'danger');
      }
    }
    if (props.editing) {
      getTodo();
    }
    return {
      todo,
      loading,
      toggleTodoStatus,
      moveTodoListPage,
      onSave,
      todoUpdated,
      showToast,
      toastMessage,
      toastAlertType,
      subjectError,
    }
  },
}
</script>
<style scoped>
.text-red {
  color: red;
}

.form-group {
  margin-bottom: 15px;
}

</style>