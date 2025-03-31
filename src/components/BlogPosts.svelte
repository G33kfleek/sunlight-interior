<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
  
    const blogPosts = writable([]);
    const apiKey = "AIzaSyAzRE-6fJNnOOTP8U99Q1h4JBLjw6xEq6E"; // Use environment variable
    const blogId = "18355884775806403968"; // Use environment variable
  
    onMount(async () => {
      try {
        const response = await fetch(`https://www.googleapis.com/blogger/v3/blogs/${blogId}/posts?key=${apiKey}`);
        const data = await response.json();
        const postsWithThumbnails = data.items?.map(post => {
          const imageMatch = post.content.match(/<img[^>]+src="([^">]+)"/);
          const thumbnail = imageMatch ? imageMatch[1] : null;
          return { ...post, thumbnail, expanded: false };
        }) || [];
        blogPosts.set(postsWithThumbnails);
      } catch (error) {
        console.error('Error fetching blog posts:', error);
      }
    });
  
    function toggleExpand(post) {
      post.expanded = !post.expanded;
      blogPosts.update(posts => posts.map(p => p.id === post.id ? post : p));
    }
  </script>
  
  <style>
    .card {
      background: rgba(77, 77, 77, 0.014);
      border-radius: 16px;
      margin-top: 1.4rem;
      overflow: hidden;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
      backdrop-filter: blur(10px);
      color: white;
      transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    }
    .card:hover {
      transform: scale(1.05);
      box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
      z-index: 1000;
      position: relative;
    }
    .thumbnail {
      width: 100%;
      height: 200px;
      object-fit: cover;
      filter: blur(10px);
      transition: filter 0.3s ease-in-out;
    }
    .thumbnail.loaded {
      filter: blur(0);
    }
    .content {
      padding: 16px;
      position: relative;
    }
    .title {
      font-size: 1.5rem;
      margin-bottom: 8px;
    }
    .timestamp {
      font-size: 0.875rem;
      color: #999;
    }
    .expand {
      cursor: pointer;
      color: #1e90ff;
      margin-top: 16px;
      display: inline-block;
    }
    .overlay {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 50%;
      background: linear-gradient(0deg, rgba(0, 0, 0, 0.8), transparent);
      transition: opacity 0.3s ease-in-out;
    }
    .card:hover .overlay {
      opacity: 0;
      z-index: 1000;
    }
    .skeleton {
      background: #333;
      width: 100%;
      height: 200px;
    }
  </style>
  
  <main class="min-h-screen p-8 bg-gray-400 text-white rounded-3xl">
    <h1 class="text-3xl font-bold mb-6">Blogs</h1>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {#each $blogPosts as post}
        <a href={post.url} target="_blank" rel="noopener noreferrer" class="card">  
          <div class="card">
            {#if post.thumbnail}
              <img 
                src={post.thumbnail} 
                alt="Thumbnail" 
                class="thumbnail" 
                on:load={e => e.target.classList.add('loaded')}
                on:error={e => e.target.src = '/path/to/default-thumbnail.jpg'}
              />
            {:else}
              <div class="skeleton"></div>
            {/if}
            <div class="overlay"></div>
            <div class="content">
              <div class="title">{post.title}</div>
              <div class="timestamp">{new Date(post.published).toLocaleDateString()}</div>
              <div 
                class="expand" 
                role="button" 
                tabindex="0" 
                on:click={() => toggleExpand(post)} 
                on:keydown={(e) => e.key === 'Enter' && toggleExpand(post)}
              >
                {post.expanded ? 'Viewed' : 'View'}
              </div>
              {#if post.expanded}
                <div class="mt-4" innerHTML={post.content}></div>
              {/if}
            </div>
          </div>
        </a>
      {/each}
    </div>
  </main>
  