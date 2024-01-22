<script>
  import { onMount, tick } from 'svelte';
  import TodoList from './lib/TodoList.svelte';
  import { v4 as uuid } from 'uuid';
  let todoList;
  let todos = null;
  let error = null;
  let isLoading = false;
  let isAdding = false;
  let disabledItems = [];
  let updateDisabled = [];
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

  const removeTodo = async (event) => {
    const id = event.detail.id;
    if (disabledItems.includes(id)) return;
    disabledItems = [...disabledItems, id];
    await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
      method: 'DELETE',
    }).then((response) => {
      if (response.ok) {
        todos = todos.filter((todo) => todo.id != event.detail.id);
      } else {
        alert('An error has occured.');
      }
    });
    disabledItems = disabledItems.filter((itemId) => itemId != id);
  };

  const handleToggleTodo = async (event) => {
    const { id, completed } = event.detail;
    if (updateDisabled.includes(id)) return;

    updateDisabled = [...disabledItems, id];
    await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
      method: 'PATCH',
      body: JSON.stringify({
        completed,
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8',
      },
    }).then(async (response) => {
      if (response.ok) {
        const updatedTodo = await response.json();
        todos = todos.map((todo) => {
          if (todo.id === id) return updatedTodo;
          return todo;
        });
      } else {
        error = 'An Error Occured';
      }
    });
    updateDisabled = updateDisabled.filter((updateId) => updateId != id);
  };
</script>

<main>
  <h3>{todos ? todos.length : 0} Todos</h3>
  <TodoList
    {todos}
    {isLoading}
    {error}
    {isAdding}
    {disabledItems}
    {updateDisabled}
    bind:this={todoList}
    on:addTodo={handleTodo}
    on:removeTodo={removeTodo}
    on:toggleTodo={handleToggleTodo}
    let:todo
    let:handleToggleTodo
    let:index
  >
    <!-- {@const { id, completed } = todo}
    <input
      class="todo-checkbox"
      type="checkbox"
      checked={completed}
      on:input={(event) => {
        event.currentTarget.checked = completed;
        handleToggleTodo(id, !completed);
      }}
    />
    <div>{todo.title}</div> -->
    <span slot="title">{index + 1}-{todo.title}</span>
  </TodoList>
  <button on:click={() => todoList.focusInput()}>Focus</button>
</main>

<style>
  :global(body) {
    background-color: rgb(48, 46, 46);
  }
</style>
