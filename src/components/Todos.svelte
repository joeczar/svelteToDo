<!-- Todos.svelte -->
<script lang="ts">
  import {alert} from "../stores"
  import FilterButton from "./FilterButton.svelte";
  import Todo from "./Todo.svelte";
  import MoreActions from "./MoreActions.svelte";
  import NewTodo from "./NewTodo.svelte";
  import TodosStatus from "./TodosStatus.svelte";
  import { Filter } from '../types/filter.enum'
  import type { TodoType } from '../types/todo.type'

  export let todos: TodoType[] = []; 

  let todoStatus: TodosStatus;

  $: newTodoId = todos.length ? Math.max(...todos.map((t) => t.id)) + 1 : 1;

  function addTodo(name: string) {
    todos = [...todos, { id: newTodoId, name, completed: false }];
    $alert = `Todo '${name}' has been added`;
  }
  function removeTodo(todo: TodoType) {
    todos = todos.filter((t) => t.id !== todo.id);
    todoStatus.focus();
    $alert = `Todo '${todo.name}'' has been deleted`;
  }
  function updateTodo(todo: TodoType) {
    const i = todos.findIndex((t) => t.id === todo.id);
    if (todos[i].name !== todo.name){
      $alert = `Todo '${todos[i].name}' has been renamed to '${todo.name}`;
    }
    if (todos[i].completed !== todo.completed)
      $alert = `Todo '${todos[i].name}' has been marked as '${
        todo.completed ? Filter.COMPLETED : Filter.ACTIVE
      }'`;
    todos[i] = { ...todos[i], ...todo };
  }

  let filter: Filter = Filter.ALL;
  const filterTodos = (filter: Filter, todos: TodoType[]) =>
    filter === Filter.ACTIVE
      ? todos.filter((t) => !t.completed)
      : filter === Filter.COMPLETED
      ? todos.filter((t) => t.completed)
      : todos;

  $: {
    if (filter === Filter.ALL) $alert = "Browsing all todos";
    else if (filter === Filter.ACTIVE) $alert = "Browsing active todos";
    else if (filter === Filter.COMPLETED) $alert = "Browsing completed todos";
  }

  const checkAllTodos = (completed: boolean) => {
    todos = todos.map((t) => ({ ...t, completed }));
    $alert = `${completed ? "Checked" : "Unchecked"} ${todos.length} todos`;
  };
  const removeCompletedTodos = () => {
    $alert = `Removed ${todos.filter((t) => t.completed).length} todos`;
    todos = todos.filter((t) => !t.completed);
  };  

  
</script>

<div class="todoapp stack-large">
  <!-- NewTodo -->
  <NewTodo autofocus on:addTodo={(e) => addTodo(e.detail)} />

  <!-- Filter -->
  <FilterButton bind:filter />

  <!-- TodosStatus -->
  <TodosStatus {todos} bind:this={todoStatus} />

  <!-- Todos -->
  <ul role="list" class="todo-list stack-large" aria-labelledby="list-heading">
    {#each filterTodos(filter, todos) as todo (todo.id)}
      <li class="todo">
        <Todo
          {todo}
          on:update={(e) => updateTodo(e.detail)}
          on:remove={(e) => removeTodo(e.detail)}
        />
      </li>
    {:else}
      <li>Nothing to do here!</li>
    {/each}
  </ul>

  <hr />

  <!-- MoreActions -->
  <MoreActions
    {todos}
    on:checkall={(e) => checkAllTodos(e.detail)}
    on:removeCompleted={removeCompletedTodos}
  />
</div>
