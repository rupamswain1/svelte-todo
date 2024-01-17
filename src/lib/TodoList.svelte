<script>
  export let todos = [];
</script>

<ul>
  {#each todos as { id, name }, index (id)}
    {@const number = index + 1}
    <li>{number}-{name}</li>
  {/each}
</ul>

<style>
</style>
