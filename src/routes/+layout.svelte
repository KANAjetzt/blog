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
  <div class="flex flex-col flex-grow w-full">
    <header
      class="flex items-center justify-center w-full py-4 lg:pb-8 backdrop-brightness-90 backdrop-blur-lg"
    >
      <div class="flex items-center justify-between w-full max-w-3xl px-4 py-2 lg:px-0 lg:py-0">
        <a
          class="text-lg font-bold sm:text-2xl !text-transparent bg-clip-text bg-gradient-to-r from-teal-500 to-teal-400 dark:to-teal-400"
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
      </div>
    </header>
    <main
      class="flex flex-col flex-grow w-full mx-auto max-w-3xl px-4 py-2 lg:px-0 lg:py-0"
      class:max-w-2xl={!$page.data.layout?.fullWidth}
    >
      <slot />
    </main>
    <footer
      class="flex items-center justify-center w-full py-4 lg:pb-8 px-4 backdrop-brightness-90 backdrop-blur-lg"
    >
      <p class="flex text-sm gap-1">
        <span class="hidden md:inline">Created using this awesome template -</span>
        <svg viewBox="0 0 24 24" aria-hidden="true" class="w-5 h-6 transition fill-teal-500">
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="M12 2C6.475 2 2 6.588 2 12.253c0 4.537 2.862 8.369 6.838 9.727.5.09.687-.218.687-.487 0-.243-.013-1.05-.013-1.91C7 20.059 6.35 18.957 6.15 18.38c-.113-.295-.6-1.205-1.025-1.448-.35-.192-.85-.667-.013-.68.788-.012 1.35.744 1.538 1.051.9 1.551 2.338 1.116 2.912.846.088-.666.35-1.115.638-1.371-2.225-.256-4.55-1.14-4.55-5.062 0-1.115.387-2.038 1.025-2.756-.1-.256-.45-1.307.1-2.717 0 0 .837-.269 2.75 1.051.8-.23 1.65-.346 2.5-.346.85 0 1.7.115 2.5.346 1.912-1.333 2.75-1.05 2.75-1.05.55 1.409.2 2.46.1 2.716.637.718 1.025 1.628 1.025 2.756 0 3.934-2.337 4.806-4.562 5.062.362.32.675.936.675 1.897 0 1.371-.013 2.473-.013 2.82 0 .268.188.589.688.486a10.039 10.039 0 0 0 4.932-3.74A10.447 10.447 0 0 0 22 12.253C22 6.588 17.525 2 12 2Z"
          />
        </svg>
        <a
          class="flex gap-1 text-sm text-teal-500"
          href="https://github.com/mattjennings/sveltekit-blog-template"
          >mattjennings/sveltekit-blog-template
        </a>
      </p>
    </footer>
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
