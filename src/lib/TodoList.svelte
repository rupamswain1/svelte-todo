<svelte:options immutable={true} />

<script>
  import Button from './Button.svelte';
  import { createEventDispatcher } from 'svelte';
  export let todos = [];

  let inputText = '';

  const dispatch = createEventDispatcher();

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

<ul>
  {#each todos as { id, name, completed }, index (id)}
    {@const number = index + 1}

    <li>
      <input
        type="checkbox"
        checked={completed}
        on:input={(event) => {
          event.currentTarget.checked = completed;
          handleToggleTodo(id, !completed);
        }}
      />
      {number}-{name}
      <button on:click={() => handleRemoveTodo(id)}>X</button>
    </li>
  {/each}
</ul>

<form class="add-todo-form" on:submit|preventDefault={handleTodo}>
  <input bind:value={inputText} />
  <Button type="submit" disabled={!inputText}>Add</Button>
</form>

<style>
</style>
