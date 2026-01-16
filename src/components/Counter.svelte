<script>
  let count = $state(0);
</script>

<section>
  <div class="inner">
    <button onclick={() => count++}>
      Count: {count}
    </button>
  </div>
</section>

<style lang="scss">
  button {
    padding: 0.5rem 1rem;
    background-color: $primary;
    color: $white;
    border-radius: 0.5rem;
    cursor: pointer;
    transition: background-color 150ms ease;

    &:hover {
      background-color: darken($primary, 10%);
    }
  }
</style>
