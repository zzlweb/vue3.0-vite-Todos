<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autofocus
        v-model="todoName"
        @keyup.enter="addTodo"
      />
    </header>
    <!-- This section should be hidden by default and shown when there are todos -->
    <section class="main">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="checkAll"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <!-- These are here just to show the structure of the list items -->
        <!-- List items should get the class `editing` when editing and `completed` when marked as completed -->
        <li
          v-for="item in list"
          :key="item.id"
          :class="{ completed: item.done, editing: item.id === currentId }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="item.done" />
            <label @dblclick="showTodo(item.id, item.name)">{{
              item.name
            }}</label>
            <button class="destroy" @click="del(item.id)"></button>
          </div>
          <input
            class="edit"
            v-model="currentName"
            @keyup.esc="currentId = ''"
            @keyup.enter="changeTodo(item)"
          />
        </li>
      </ul>
    </section>
    <!-- This footer should hidden by default and shown when there are todos -->
    <footer class="footer">
      <!-- This should be `0 items left` by default -->
      <span class="todo-count"
        ><strong>{{ leftCount }}</strong> item left</span
      >
      <!-- Remove this if you don't implement routing -->
      <ul class="filters">
        <li>
          <a class="selected" href="#/">All</a>
        </li>
        <li>
          <a href="#/active">Active</a>
        </li>
        <li>
          <a href="#/completed">Completed</a>
        </li>
      </ul>
      <!-- Hidden if no completed items are left ↓ -->
      <button class="clear-completed" v-if="isShow" @click="clearTodo">
        Clear completed
      </button>
    </footer>
  </section>
</template>

<script>
import { reactive, onMounted, toRefs, computed ,watch} from "vue";
export default {
  setup() {
    const state = reactive({
      // todo list 数据
      list: JSON.parse(localStorage.getItem('todos'))  || [] ,
      todoName: "",
      // 用于记录当前双击的item id
      currentId: "",
      // 用记录当前被双击的item name
      currentName: "",
    });

    const del = (id) => {
      state.list = state.list.filter((item) => item.id !== id);
    };

    const addTodo = () => {
      state.list.push({
        id: new Date(),
        name: state.todoName,
        done: false,
      });

      state.todoName = "";
    };

    const showTodo = (id, name) => {
      console.log(name);
      state.currentId = id;
      state.currentName = name;
    };

    const changeTodo = (item) => {
      item.name = state.currentName;
      state.currentId = "";
    };

    const clearTodo = () => {
      state.list = state.list.filter((item) => !item.done);
    };

    const computedDate = reactive({
      leftCount: computed(() => {
        return state.list.filter((item) => !item.done).length;
      }),

      checkAll: computed({
        get: () => {
          return state.list.every((item) => item.done);
        },
        set: (value) => {
          return state.list.forEach((item) => {
            item.done = value;
          });
        },
      }),

      isShow: computed(() => {
        return state.list.some((item) => item.done);
      }),
    });

    // 监听属性
    watch(
      // 第一个参数为是一个函数或者是一个ref(响应式数据)
      () => state.list, (value, oldValue) => {
        localStorage.setItem('todos', JSON.stringify(state.list))
      },
      {
        // 深度监听
        deep: true
      }
    )
    return {
      ...toRefs(state),
      ...toRefs(computedDate),
      del,
      addTodo,
      showTodo,
      changeTodo,
      clearTodo
    };
  },
};
</script>
