<svelte:options immutable={true} />

<script>
  import Button from './Button.svelte';
  import { afterUpdate, beforeUpdate, createEventDispatcher } from 'svelte';
  import { crossfade, fade, scale } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  const [send, receive] = crossfade({
    duration: 400,
    fallback(node) {
      return scale(node, { start: 0.5, duration: 300 });
    },
  });

  export let todos = null;
  export let error = null;
  export let isLoading = false;
  export let isAdding = false;
  export let disabledItems = [];
  export let updateDisabled = [];
  export let scrollOnAdd = undefined;
  let inputText = '';
  let input;
  let listDiv;
  let prevTodos = todos;
  let autoscroll = false;
  let done = null;
  let todo = null;
  //Lifecycles
  afterUpdate(() => {
    if (scrollOnAdd) {
      let pos;
      if (scrollOnAdd === 'top') pos = 0;
      if (scrollOnAdd === 'bottom') pos = listDiv.scrollHeight;
      if (autoscroll) {
        listDiv.scrollTo(0, pos);
        autoscroll = false;
      }
    }
  });

  $: {
    autoscroll = todos && prevTodos && todos.length > prevTodos.length;
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
  $: done = todos ? todos.filter((t) => t.completed) : [];

  $: todo = todos ? todos.filter((t) => !t.completed) : [];
</script>

<div bind:this={listDiv} class="todo-container">
  {#if isLoading}
    <div class="no-item-text">
      <p>Loading . . .</p>
    </div>
  {:else if error}
    <div class="no-item-text">
      <p>{error}</p>
    </div>
  {:else if !todos || todos?.length === 0}
    <div class="no-item-text">
      <p>No Todo Items are available!!</p>
    </div>
  {:else}
    <div style class="flex">
      {#each [todo, done] as list, index}
        <div>
          <h2>{index === 0 ? 'Todo' : 'Done'}</h2>

          <ul>
            {#each list as todo, index (todo.id)}
              {@const number = index + 1}
              {@const { id, title, completed } = todo}
              <li animate:flip={{ duration: 300 }}>
                <slot {todo} {handleToggleTodo} {index}>
                  <div
                    class="todo-item"
                    class:todo-completed={completed}
                    class:item-disabled={disabledItems.includes(id) ||
                      updateDisabled.includes(id)}
                    in:receive={{ key: id }}
                    out:send={{ key: id }}
                  >
                    <!-- <div
                      class="todo-item"
                      class:todo-completed={completed}
                      class:item-disabled={disabledItems.includes(id) ||
                        updateDisabled.includes(id)}
                      transition:scale|local={{ start: 0.5, duration: 300 }}
                    > -->
                    <input
                      class="todo-checkbox"
                      type="checkbox"
                      checked={completed}
                      on:input={(event) => {
                        event.currentTarget.checked = completed;
                        handleToggleTodo(id, !completed);
                      }}
                    />
                    <slot name="title">
                      <span
                        class="todo-label"
                        class:todo-completed-label={completed}
                      >
                        {number}-{title}
                      </span>
                    </slot>
                    <button
                      class="todo-remove-btn"
                      on:click={() => handleRemoveTodo(id)}>x</button
                    >
                  </div>
                </slot>
              </li>
            {/each}
          </ul>
        </div>
      {/each}
    </div>
  {/if}
</div>
<form class="add-todo-form" on:submit|preventDefault={handleTodo}>
  <input
    bind:value={inputText}
    bind:this={input}
    placeholder="Enter Todo Name"
  />
  <Button type="submit" disabled={!inputText || isAdding}>Add</Button>
</form>

<style>
  .flex {
    display: flex;
  }
  .todo-container {
    max-height: 250px;
    overflow: auto;
    background-color: rgb(70, 68, 68);
    /* width: 50%; */
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
  .todo-completed-label {
    text-decoration: line-through;
    opacity: 0.3;
  }
  .item-disabled {
    opacity: 0.1;
    cursor: not-allowed;
  }
</style>
