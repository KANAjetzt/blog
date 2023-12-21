<script>
  import '../app.css'
  import '../prism.css'
  import { Icon, Moon, Sun } from 'svelte-hero-icons'
  import { browser } from '$app/environment'
  import { name } from '$lib/info'
  import { page } from '$app/stores'

  let isDarkMode = browser ? Boolean(document.documentElement.classList.contains('dark')) : true

  function disableTransitionsTemporarily() {
    document.documentElement.classList.add('[&_*]:!transition-none')
    window.setTimeout(() => {
      document.documentElement.classList.remove('[&_*]:!transition-none')
    }, 0)
  }
</script>

<div class="flex flex-col min-h-screen">
  <div class="flex flex-col flex-grow w-full px-4 py-2">
    <header class="flex items-center justify-between w-full max-w-3xl py-4 mx-auto lg:pb-8">
      <a
        class="text-lg font-bold sm:text-2xl !text-transparent bg-clip-text bg-gradient-to-r from-teal-500 to-teal-600 dark:to-teal-400"
        href="/"
      >
        {name}
      </a>

      <button
        type="button"
        role="switch"
        aria-label="Toggle Dark Mode"
        aria-checked={isDarkMode}
        class="w-5 h-5 sm:h-8 sm:w-8 sm:p-1"
        on:click={() => {
          isDarkMode = !isDarkMode
          localStorage.setItem('isDarkMode', isDarkMode.toString())

          disableTransitionsTemporarily()

          if (isDarkMode) {
            document.querySelector('html').classList.add('dark')
          } else {
            document.querySelector('html').classList.remove('dark')
          }
        }}
      >
        <Icon src={Sun} class="hidden text-zinc-500 dark:block" />
        <Icon src={Moon} class="block text-zinc-400 dark:hidden" />
      </button>
    </header>
    <main
      class="flex flex-col flex-grow w-full mx-auto"
      class:max-w-2xl={!$page.data.layout?.fullWidth}
    >
      <slot />
    </main>
  </div>
</div>

<style>
  :global(:root) {
    --scrollbar-color-primary: hsla(0, 0%, 36%, 0.466);
    --scrollbar-color-secondary: hsla(0, 0%, 85%, 0.25);
    --scrollbar-color-secondary-a-50: hsla(0, 0%, 85%, 0.5);
  }

  /* --- Custom Scrollbar for Codeblocks --- */
  :global(pre) {
    /* Foreground, Background */
    scrollbar-color: var(--scrollbar-color-secondary) var(--scrollbar-color-primary);
  }
  :global(pre::-webkit-scrollbar) {
    width: 10px; /* Mostly for vertical scrollbars */
    height: 10px; /* Mostly for horizontal scrollbars */
  }
  :global(pre::-webkit-scrollbar-thumb) {
    /* Foreground */
    border-radius: 10px;
    background-color: var(--scrollbar-color-secondary);
    background-image: -webkit-linear-gradient(
      90deg,
      transparent,
      --scrollbar-color-secondary-a-50,
      transparent,
      transparent
    );
  }

  :global(pre::-webkit-scrollbar-track) {
    /* Background */
    margin: 10px;
    background: var(--scrollbar-color-primary);
    border-radius: 10px;
  }
</style>
