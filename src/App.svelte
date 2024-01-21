<script>
  import { onMount, tick } from 'svelte';
  import TodoList from './lib/TodoList.svelte';
  import { v4 as uuid } from 'uuid';
  let todoList;
  let todos = null;
  let error = null;
  let isLoading = false;
  let isAdding = false;
  async function loadTodos() {
    isLoading = true;
    fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(
      async (response) => {
        if (response.ok) {
          todos = await response.json();
        } else {
          error = 'An Error has occured';
        }
        isLoading = false;
      }
    );
  }

  onMount(() => {
    loadTodos();
  });

  const handleTodo = async (event) => {
    event.preventDefault();
    isAdding = true;
    fetch('https://jsonplaceholder.typicode.com/todos', {
      method: 'POST',
      body: JSON.stringify({
        title: event.detail.title,
        completed: false,
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8',
      },
    }).then(async (response) => {
      if (response.ok) {
        const todo = await response.json();
        todos = [
          ...todos,
          {
            id: uuid(),
            title: event.detail.title,
            completed: false,
          },
        ];
      } else {
        alert('An error has occured.');
      }
      isAdding = false;
      await tick();
      todoList.clearInput();
      todoList.focusInput();
    });
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
</script>

<main>
  <h3>{todos ? todos.length : 0} Todos</h3>
  <TodoList
    {todos}
    {isLoading}
    {error}
    {isAdding}
    bind:this={todoList}
    on:addTodo={handleTodo}
    on:removeTodo={removeTodo}
    on:toggleTodo={handleToggleTodo}
  />
  <button on:click={() => todoList.focusInput()}>Focus</button>
</main>

<style>
  :global(body) {
    background-color: rgb(48, 46, 46);
  }
</style>
