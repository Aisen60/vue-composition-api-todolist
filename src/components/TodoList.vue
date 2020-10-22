<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autofocus
        v-model="inputValue"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main">
      <input id="toggle-all" class="toggle-all" type="checkbox" />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id + '-' + todo.title"
          class="todo"
          :class="{ completed: todo.completed }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label>{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" :value="todo.title" />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length !== 0">
      <span class="todo-count">
        <strong>{{ todos.length }}</strong>
        {{ todos.length > 1 ? "items" : "item" }}
        left
      </span>
      <ul class="filters">
        <li>
          <a :class="{ selected: type === 'all' }" href="#/all">All</a>
        </li>
        <li>
          <a :class="{ selected: type === 'active' }" href="#/active">Active</a>
        </li>
        <li>
          <a :class="{ selected: type === 'completed' }" href="#/completed"
            >Completed</a
          >
        </li>
      </ul>
      <button
        class="clear-completed"
        v-show="completedCount.length > 0"
        @click="removeCompleted"
      >
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <p>Created by <a href="http://sindresorhus.com">Aisen60</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</template>

<script>
import {
  ref,
  reactive,
  computed,
  watchEffect,
  onMounted,
  onUnmounted,
} from "vue";
import { getStorage, setStorage } from "../utils/";
const useAdd = (todos) => {
  const inputValue = ref("");
  const addTodo = () => {
    const title = inputValue.value && inputValue.value.trim();
    if (title.length === 0) return;
    todos.push({
      id: Math.round(new Date()),
      title,
      completed: false,
    });
    inputValue.value = "";
  };
  return {
    inputValue,
    addTodo,
  };
};

const useRemove = (todos) => {
  const removeTodo = (todo) => {
    const index = todos.indexOf(todo);
    todos.splice(index, 1);
  };

  const removeCompleted = () => {
    todos.value = todos.value.filter((todo) => !todo.completed);
  };
  return {
    removeTodo,
    removeCompleted,
  };
};

const useFilter = (todos) => {
  const type = ref("all");

  const filter = {
    all: (list) => list,
    active: (list) => list.filter((todo) => !todo.completed),
    completed: (list) => list.filter((todo) => todo.completed),
  };

  const handlerHashChange = () => {
    const hash = window.location.hash.replace("#/", "");
    if (filter[hash]) {
      type.value = hash;
      filter[hash](todos);
    } else {
      type.value = "all";
      window.location.hash = "";
    }
  };

  onMounted(() => {
    window.addEventListener("hashchange", handlerHashChange);
    handlerHashChange();
  });

  onUnmounted(() => {
    window.removeEventListener("hashchange");
  });

  const filteredTodos = computed(() => filter[type.value](todos.value));

  const completedCount = computed(() => filter.completed(todos.value));

  return { type, filteredTodos, completedCount };
};

const useStorage = () => {
  const storageKey = "TODOLIST",
    todos = ref(JSON.parse(getStorage(storageKey)) || []);
  console.log(todos.value);
  watchEffect(() => {
    setStorage(storageKey, JSON.stringify(todos.value));
  });
  return todos;
};

export default {
  name: "TodoList",
  setup() {
    const todos = useStorage();
    const { removeTodo, removeCompleted } = useRemove(todos);

    return {
      todos,
      removeTodo,
      removeCompleted,
      ...useAdd(todos),
      // ...useRemove(todos),
      ...useFilter(todos),
    };
  },
};
</script>
