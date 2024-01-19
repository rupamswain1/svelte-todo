<svelte:options immutable={true} />

<script>
  import Button from './Button.svelte';
  import { afterUpdate, beforeUpdate, createEventDispatcher } from 'svelte';
  export let todos = [];

  let inputText = '';
  let input;
  let listDiv;
  let prevTodos = todos;
  let autoscroll = false;
  //Lifecycles
  afterUpdate(() => {
    if (autoscroll) {
      listDiv.scrollTo(0, listDiv.scrollHeight);
      autoscroll = false;
    }
  });

  $: {
    autoscroll = todos.length > prevTodos.length;
    prevTodos = todos;
  }

  const dispatch = createEventDispatcher();

  export const clearInput = () => {
    inputText = '';
  };

  export const focusInput = () => {
    input.focus();
  };

  const handleTodo = () => {
    const isNotCancelled = dispatch(
      'addTodo',
      {
        title: inputText,
      },
      { cancelable: true }
    );
    if (isNotCancelled) inputText = '';
  };

  const handleRemoveTodo = (id) => {
    dispatch('removeTodo', { id });
  };

  const handleToggleTodo = (id, state) => {
    dispatch('toggleTodo', {
      id,
      completed: state,
    });
  };
</script>

<div bind:this={listDiv} class="todo-container">
  {#if todos.length === 0}
    <div class="no-item-text">
      <p>No Todo Items are available!!</p>
    </div>
  {:else}
    <ul>
      {#each todos as { id, name, completed }, index (id)}
        {@const number = index + 1}

        <li>
          <div class="todo-item" class:completed={'todo-completed'}>
            <input
              class="todo-checkbox"
              type="checkbox"
              checked={completed}
              on:input={(event) => {
                event.currentTarget.checked = completed;
                handleToggleTodo(id, !completed);
              }}
            />
            <span class="todo-label">
              {number}-{name}
            </span>

            <button
              class="todo-remove-btn"
              on:click={() => handleRemoveTodo(id)}>x</button
            >
          </div>
        </li>
      {/each}
    </ul>
  {/if}
</div>
<form class="add-todo-form" on:submit|preventDefault={handleTodo}>
  <input
    bind:value={inputText}
    bind:this={input}
    placeholder="Enter Todo Name"
  />
  <Button type="submit" disabled={!inputText}>Add</Button>
</form>

<style>
  .todo-container {
    max-height: 250px;
    overflow: auto;
    background-color: rgb(70, 68, 68);
    width: 50%;
    border-radius: 5px;
    margin-bottom: 10px;
  }
  ul {
    list-style: none;
    padding: 0;
  }
  .todo-item {
    cursor: pointer;
    color: white;
    border: 1px solid white;
    margin: 10px;
    border-radius: 5px;
    display: flex;
    flex-direction: row;
    align-items: baseline;
    padding: 5px;
    background-color: rgb(116, 116, 116);
  }
  .todo-label {
    margin-left: 5px;
  }

  .todo-remove-btn {
    margin-left: auto;
    margin-bottom: 0;
    width: 23px;
    height: 23px;
    background-color: rgb(247, 14, 14);
    border: none;
    color: white;
    padding: 0;
    border-radius: 50%;
    line-height: 0.2;
    font-weight: 600;
    cursor: pointer;
    display: none;
  }
  .todo-item:hover .todo-remove-btn {
    display: block;
  }
  .no-item-text {
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .add-todo-form {
    background-color: rgb(70, 68, 68);
    width: 48%;
    border-radius: 5px;
    padding: 10px;
    display: flex;
    align-items: center;
  }
  .add-todo-form input {
    margin: 0;
    flex: 1;
    margin-right: 10px;
    border-radius: 5px;
  }
</style>
