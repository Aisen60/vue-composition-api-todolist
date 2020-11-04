<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autofocus
        v-model="input"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main" v-show="count">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="allDone"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :class="{ editing: todo === editingTodo, completed: todo.completed }"
          :key="todo"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="remove(todo)"></button>
          </div>
          <input
            class="edit"
            type="text"
            v-editing-focus="todo === editingTodo"
            v-model="todo.text"
            @keyup.enter="doneEdit(todo)"
            @blur="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="count">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong>
        {{ remainingCount > 1 ? "items" : "item" }} left
      </span>
      <ul class="filters">
        <li>
          <a href="#/all" :class="{ selected: type === 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: type === 'active' }">Active</a>
        </li>
        <li>
          <a href="#/completed" :class="{ selected: type === 'completed' }">
            Completed
          </a>
        </li>
      </ul>
      <button
        class="clear-completed"
        @click="removeCompleted"
        v-show="count > remainingCount"
      >
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <p>Created by <a href="https://github.com/Aisen60">Aisen60</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</template>

<script>
import "./assets/index.css";
import { ref, computed, onMounted, onUnmounted, watchEffect } from "vue";
import useLocalStorage from "./utils/useLocalStorage";
const storage = useLocalStorage();

const useAdd = (todos) => {
  const input = ref("");
  const addTodo = () => {
    const text = input.value && input.value.trim();
    if (text.length === 0) return;
    todos.value.push({
      text,
      completed: false,
    });
    input.value = "";
  };
  return {
    input,
    addTodo,
  };
};

const useRemove = (todos) => {
  const remove = (todo) => {
    const index = todos.value.indexOf(todo);
    todos.value.splice(index, 1);
  };
  const removeCompleted = () => {
    todos.value = todos.value.filter((todo) => !todo.completed);
  };
  return {
    remove,
    removeCompleted,
  };
};

const useEdit = (remove) => {
  let beforeEditingText = "";
  const editingTodo = ref(null);
  const editTodo = (todo) => {
    beforeEditingText = todo.text;
    editingTodo.value = todo;
  };
  const doneEdit = (todo) => {
    if (!editingTodo.value) return;
    todo.text = todo.text.trim();
    todo.text || remove(todo);
    editingTodo.value = null;
  };
  const cancelEdit = (todo) => {
    editingTodo.value = null;
    todo.text = beforeEditingText;
  };
  return {
    editingTodo,
    editTodo,
    doneEdit,
    cancelEdit,
  };
};

const useFilter = (todos) => {
  const allDone = computed({
    get() {
      return !todos.value.filter((todo) => !todo.completed).length;
    },
    set(value) {
      todos.value.forEach((todo) => {
        todo.completed = value;
      });
    },
  });

  const filter = {
    all: (list) => list,
    active: (list) => list.filter((todo) => !todo.completed),
    completed: (list) => list.filter((todo) => todo.completed),
  };

  const type = ref("all");

  const filteredTodos = computed(() => filter[type.value](todos.value));

  const remainingCount = computed(() => filter["active"](todos.value).length);

  const count = computed(() => todos.value.length);

  const onHashChange = () => {
    const hash = window.location.hash.replace("#/", "");
    if (filter[hash]) {
      type.value = hash;
    } else {
      type.value = "all";
      window.location.hash = "";
    }
  };

  onMounted(() => {
    window.addEventListener("hashchange", onHashChange);
    onHashChange();
  });

  onUnmounted(() => {
    window.removeEventListener("hashchange", onHashChange);
  });

  return {
    allDone,
    count,
    type,
    filteredTodos,
    remainingCount,
  };
};

const useStorage = () => {
  const KEY = "TODOKEYS";
  const todos = ref(storage.getItem(KEY) || []);
  watchEffect(() => {
    storage.setItem(KEY, todos.value);
  });
  return todos;
};

export default {
  name: "TodoList",
  setup() {
    const todos = useStorage();
    const { remove, removeCompleted } = useRemove(todos);
    return {
      todos,
      remove,
      removeCompleted,
      ...useAdd(todos),
      ...useEdit(remove),
      ...useFilter(todos),
    };
  },
  directives: {
    editingFocus: (el, binding) => {
      binding.value && el.focus();
    },
  },
};
</script>
