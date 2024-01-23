<script>
  import { onMount, tick } from 'svelte';
  import TodoList from './lib/TodoList.svelte';
  import { v4 as uuid } from 'uuid';
  import { fly } from 'svelte/transition';
  import spin from './lib/transitions/spin';
  let todoList;
  let todos = null;

  let error = null;
  let isLoading = false;
  let isAdding = false;
  let disabledItems = [];
  let updateDisabled = [];
  let showTodo = true;
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
          {
            id: uuid(),
            title: event.detail.title,
            completed: false,
          },
          ...todos,
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
  <div>
    <input
      type="checkbox"
      checked={showTodo}
      on:input={() => (showTodo = !showTodo)}
    />
    <span style:color="white">Show hide Todo</span>
  </div>
  {#if showTodo}
    <div transition:spin={{ spin: 1, duration: 500 }} style="opacity=0.5">
      <h3 style="color: white;">
        {#key todos?.length}
          <span style:display={'inline-block'} in:fly|local={{ y: -10 }}
            >{todos ? todos.length : 0}</span
          >{/key} Todos
      </h3>
      <TodoList
        {todos}
        {isLoading}
        {error}
        {isAdding}
        {disabledItems}
        {updateDisabled}
        scrollOnAdd={'top'}
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
        <!-- <span slot="title">{index + 1}-{todo.title}</span> -->
      </TodoList>

      <button on:click={() => todoList.focusInput()}>Focus</button>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    background-color: rgb(48, 46, 46);
    color: white;
  }
</style>
