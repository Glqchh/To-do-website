<script setup>
import { ref, onMounted, computed, watch } from 'vue';

const todos = ref([]);
const name = ref('');
const input_content = ref('');

const todos_asc = computed(() =>
  todos.value.sort((a, b) => a.createdAt - b.createdAt)
);

const autoResize = (event) => {
  const textarea = event.target;
  textarea.style.height = 'auto'; // Reset height to calculate new height
  textarea.style.height = `${textarea.scrollHeight}px`; // Set height to match content
};

watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});

watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal));
}, {
  deep: true,
});

const addTodo = () => {
  if (input_content.value.trim() === '') {
    return;
  }

  todos.value.push({
    content: input_content.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime(),
  });

  input_content.value = '';
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  todos.value = JSON.parse(localStorage.getItem('todos')) || [];
});
</script>

<template>
  <main class="app">
    <section class="create-todo">
      <h3>Create a To-Do Item</h3>
      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your mind today?</h4>
        <input 
          type="text" 
          name="content" 
          id="content" 
          placeholder="e.g. Make a video" 
          v-model="input_content" 
        />
        <input type="submit" value="Add" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TO-DO LIST</h3>
      <div class="list" id="todo-list">
        <div 
          v-for="todo in todos_asc" 
          :key="todo.createdAt" 
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
          </label>
          <div class="todo-content">
             <textarea 
              v-model="todo.content"
              :readonly="!todo.editable"
               @input="autoResize($event)"
               @dblclick="todo.editable = true"
               @blur="todo.editable = false"
            ></textarea>
        </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
