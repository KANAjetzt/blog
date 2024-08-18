<script>
  import { fade } from 'svelte/transition'

  let show_button = true
  let show_game = false

  export let button_text = ''
</script>

{#if show_button}
  <button
    transition:fade={{ duration: 300 }}
    class="rounded-3xl p-5 sm:p-5 sm:m-auto bg-teal-500 text-zinc-900 text-base font-semibold text-center"
    on:click={() => {
      show_button = false
      setTimeout(() => {
        show_game = true
      }, 300)
    }}
  >
    {button_text}
  </button>
{/if}

{#if show_game}
  <div transition:fade={{ duration: 1000 }}>
    <slot />
  </div>
  <button
    class="rounded-3xl p-5 sm:p-5 sm:m-auto text-teal-500 text-base font-semibold text-center"
    on:click={() => {
      show_game = false
      setTimeout(() => {
        show_button = true
      }, 1000)
    }}>close</button
  >
{/if}
