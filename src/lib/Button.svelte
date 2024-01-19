<script>
  export let type = '';
  export let disabled = false;
  export let handleClick = () => {};
</script>

<button
  class={`btn primary ${disabled && 'btn-disable'}`}
  {type}
  {disabled}
  on:click={handleClick}
>
  <slot />
</button>

<style>
  .btn {
    background-color: #df7015;
    color: white;
    border-radius: 5px;
    width: 150px;
    font-weight: bold;
    cursor: pointer;
    border: 1px solid black;
    margin: 0;
  }
  .btn:hover {
    background-color: #f07943;
  }
  .btn-disable,
  .btn-disable:hover {
    background-color: #8a4105;
  }
</style>
