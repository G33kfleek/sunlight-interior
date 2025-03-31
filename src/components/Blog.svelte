<script>
  export let posts = [];
</script>

<section id="blog" class="p-10">
  <h2 class="text-4xl mb-5">Latest Blog Posts</h2>
  {#each posts as post}
    <article class="mb-5 border-b pb-5">
      <h3 class="text-2xl font-bold">
        <a href={post.link} target="_blank" class="text-blue-500 hover:underline">{post.title}</a>
      </h3>
      <p class="text-gray-600">{new Date(post.published).toLocaleDateString()}</p>
      <div class="mt-2 text-lg">{@html post.content}</div>
    </article>
  {/each}
</section>