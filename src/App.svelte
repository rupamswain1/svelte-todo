<script>
  import TodoList from './lib/TodoList.svelte';
  import { v4 as uuid } from 'uuid';

  let todos = [
    {
      id: uuid(),
      name: 'Todo 1',
      completed: true,
    },
    {
      id: uuid(),
      name: 'Todo 2',
      completed: true,
    },
    {
      id: uuid(),
      name: 'Todo 3',
      completed: false,
    },
  ];

  const handleTodo = (event) => {
    event.preventDefault();
    todos = [
      ...todos,
      {
        id: uuid(),
        name: event.detail.title,
        completed: false,
      },
    ];
  };

  const removeTodo = (event) => {
    todos = todos.filter((todo) => todo.id != event.detail.id);
  };

  const handleToggleTodo = (event) => {
    const { id, completed } = event.detail;
    todos = todos.map((todo) => {
      if (todo.id === id) return { ...todo, completed };
      return todo;
    });
  };
  $: console.log(todos);
</script>

<main>
  <h3>{todos.length} Todos</h3>
  <TodoList
    {todos}
    on:addTodo={handleTodo}
    on:removeTodo={removeTodo}
    on:toggleTodo={handleToggleTodo}
  />
</main>

<style>
</style>
