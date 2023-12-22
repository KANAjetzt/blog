<script>
  import PostPreview from '$lib/components/PostPreview.svelte'
  import PostDate from '$lib/components/PostDate.svelte'

  export let posts

  console.log(posts)
</script>

<div class="flex flex-col gap-16 md:border-l md:border-zinc-100 md:pl-6 md:dark:border-zinc-700/40">
  {#each posts as post}
    <article class="grid items-start grid-cols-4 gap-8">
      <PostDate class="flex-col hidden md:flex text-sm" {post} />

      <div class="col-span-2 md:col-span-2">
        <PostPreview {post}>
          <slot slot="eyebrow">
            <PostDate class="md:hidden" {post} collapsed decorate />
          </slot>
        </PostPreview>
      </div>
      {#if post.cover}
        <div class="rounded-3xl h-full w-full overflow-hidden">
          <a href={`/post/${post.slug}`} data-sveltekit-prefetch>
            <img
              class="rounded-3xl h-full w-full object-cover hover:scale-110 transition-transform duration-300"
              src={`/imgs/${post.slug}/${post.preview_image}`}
              alt={post.cover_alt}
            />
          </a>
        </div>
      {/if}
    </article>
  {/each}
</div>
