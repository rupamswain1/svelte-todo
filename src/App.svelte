<script>
  import TodoList from './lib/TodoList.svelte';
  import { v4 as uuid } from 'uuid';

  const todos = [
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
      completed: true,
    },
  ];
</script>

<main>
  <TodoList {todos} />
</main>

<style>
</style>
