<script>
  import { browser } from '$app/environment'
  import { onMount } from 'svelte'
  import Card from './Card.svelte'

  export let post

  let elements = []
  let headings = post.headings
  let activeHeading = headings[0]
  let scrollY
  let is_clicked = false

  onMount(() => {
    updateHeadings()
    setActiveHeading()
  })

  function updateHeadings() {
    headings = post.headings

    if (browser) {
      elements = headings.map((heading) => {
        return document.getElementById(heading.id)
      })
    }
  }
  function setActiveHeading() {
    if (is_clicked) {
      is_clicked = false
      return
    }

    scrollY = window.scrollY

    const visibleIndex =
      elements.findIndex((element) => element.offsetTop + element.clientHeight > scrollY) - 1

    activeHeading = headings[visibleIndex]

    const pageHeight = document.body.scrollHeight
    const scrollProgress = (scrollY + window.innerHeight) / pageHeight

    if (!activeHeading) {
      if (scrollProgress > 0.5) {
        activeHeading = headings[headings.length - 1]
      } else {
        activeHeading = headings[0]
      }
    }
  }
</script>

<svelte:window on:scroll={setActiveHeading} />

<Card>
  <slot slot="description">
    <ul class="flex flex-col gap-2">
      {#each headings as heading}
        <li
          class="pl-2 transition-colors border-teal-500 heading text-zinc-500 dark:text-zinc-400 hover:text-zinc-900 dark:hover:text-zinc-100"
          class:active={activeHeading === heading}
          style={`--depth: ${
            // consider h1 and h2 at the same depth, as h1 will only be used for page title
            Math.max(0, heading.depth - 1)
          }`}
        >
          <a
            href={`#${heading.id}`}
            on:click={(e) => {
              // Prevent the scroll event from triggering a active heading update
              is_clicked = true

              const clicked_heading_id = e.target.hash.replace('#', '')
              const clicked_heading_index = headings.findIndex(
                (heading) => heading.id === clicked_heading_id
              )

              activeHeading = headings[clicked_heading_index]
            }}>{heading.value}</a
          >
        </li>
      {/each}
    </ul>
  </slot>
</Card>

<style lang="postcss">
  .heading {
    padding-left: calc(var(--depth, 0) * 0.35rem);
  }

  .active {
    @apply font-medium text-slate-900 border-l-2 -ml-[2px];
  }

  /* can't use dark: modifier in @apply */
  :global(.dark) .active {
    @apply text-slate-100;
  }
</style>
