<template>
  <form @submit.prevent="onSubmit">
    <div class="d-flex">
      <div class="flex-grow-1 mr-2">
        <input
            class="form-control"
            type="text"
            v-model="todo"
            placeholder="Type new to-do"
        >
      </div>
      <div>
        <button
            class="btn btn-primary"
            type="onSubmit"
        >
          Add!!
        </button>
      </div>
    </div>
    <div class="error" v-show="hasError">
      This field cannot be empty!!
    </div>
  </form>
</template>

<script>
import {getCurrentInstance, ref} from 'vue';

export default {
  emits: [
    'add-todo'
  ],
  setup() {
    const {emit} = getCurrentInstance();

    const todo = ref('');
    const hasError = ref(false);
    const onSubmit = () => {
      if (todo.value == '') {
        hasError.value = true;
      } else {
        emit('add-todo', {
          id: Date.now(),
          subject: todo.value,
          completed: false
        });
        hasError.value = false;
        todo.value = '';
      }
    }
    return {
      todo,
      hasError,
      onSubmit
    }
  }

}
</script>

<style>

</style>