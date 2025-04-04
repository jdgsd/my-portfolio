<script>
    import { page } from "$app/stores";
    let pages = [
        { url: "./", title: "Home" },
        { url: "./projects", title: "Projects" },
        { url: "./cv", title: "CV"},
        { url: "./contact", title: "Contact" },
        { url: "https://github.com/jdgsd", title:"GitHub"}
    ];

    let localStorage = globalThis.localStorage ?? {};
    $: localStorage.colorScheme = colorScheme;

    let colorScheme = "light dark";
    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);
    
</script>

<nav>
    {#each pages as p}
    <a
    href={p.url}
    class:current={"." + $page.route.id === p.url}
    target={p.url.startsWith("http") ? "_blank" : null}
  >
    {p.title}
  </a>  
    {/each}
</nav>

<label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="dark">Dark</option>
        <option value="light">Light</option>
    </select>
</label>

<style>
    nav ul, 
nav li {
    display: contents;
}

nav {
  display: flex;
}

nav a{
  flex: 1;
  text-align: center;
  text-decoration: none;
  color: inherit;
  padding: 0.5em;
  margin-bottom: 1rem;
  border-bottom: 1px solid var(--border-color);
  --border-color: oklch(50% 10% 200 / 40%);
}

nav a.current {
  border-bottom: 0.4em solid var(--border-color);
}

nav a:hover {
  border-bottom: 0.4em solid var(--color-accent);
  background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
}

label.color-scheme {
  position: absolute;
  top: 1rem;
  right: 1rem;
  display: inline-flex;
  gap: 4px;
  font: 80%/1 system-ui;
}
</style>


<slot />

{#await fetch("https://api.github.com/users/jdgsd")}
  <p>Loading...</p>
{:then response}
  {#await response.json()}
    <p>Decoding...</p>
  {:then data}
    <section>
      <h2>My GitHub Stats</h2>
      <dl>
        <dt>Followers:</dt>
        <dd>{data.followers}</dd>
        <dt>Following:</dt>
        <dd>{data.following}</dd>
        <dt>Public Repositories:</dt>
        <dd>{data.public_repos}</dd>
      </dl>
    </section>
  {:catch error}
    <p class="error">Something went wrong: {error.message}</p>
  {/await}
{:catch error}
  <p class="error">Something went wrong: {error.message}</p>
{/await}