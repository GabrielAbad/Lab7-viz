<script>
import "../style.css";

import { base } from '$app/paths';
import { page } from "$app/stores";

let pages = [
  { url: "/", title: "Home" },
  { url: "/projects", title: "Projects" },
  { url: "/contact", title: "Contact" },
  { url: "/resume", title: "Resume" },
  { url: "/meta", title: "Meta" },
  { url: "https://github.com/Mike-Kowalski", title: "Github" }
];

let root = globalThis?.document?.documentElement;
let localStorage = globalThis.localStorage ?? {};

let colorScheme = localStorage.colorScheme ?? "light dark";

$: localStorage.colorScheme = colorScheme;
$: root?.style.setProperty("color-scheme", colorScheme);

$: console.log(localStorage);

</script>

<nav>
  {#each pages as p}
  <a
            href={p.url.startsWith("http") 
                ? p.url 
                : `${base}${p.url}`}
            class:current={$page.route.id === p.url}
            target={p.url.startsWith("http") ? "_blank" : undefined}
        >
    {p.title}
  </a>
  {/each}
</nav>

<label class="color-scheme">
  Theme:
  <select bind:value={ colorScheme }>
    <option value="light dark"> Automatic </option>
    <option value="light"> Light </option>
    <option value="dark"> Dark </option>
</select>

</label>

<slot />
<style>
nav ul,
nav il {
	display: contents; /* step 2.1 */
}

nav {
	--border-color: oklch(50% 10% 200 / 40%);
	
	display: flex; /*step 2.2 part 1*/
	margin-bottom: 1em; /*step 2.2 part 2*/
	border-bottom-width:1px; /*step 2.2 part 3*/
	border-bottom-style:solid; /*step 2.2 part 3*/
	
	border-bottom-color: var(--border-color);
}

nav a {
	flex:1; /*step 2.2 part 1*/
	text-decoration: none;/*step 2.2 part 2*/
	color: inherit; /*step 2.2 part 2*/
	text-align: center; /*step 2.2 part 2*/
	padding: 0.5em; /*step 2.2 part 2*/
}

nav a.current {
	border-bottom-width:0.4em; /*step 2.2 task*/
	border-bottom-style:solid; /*step 2.2 task*/
	border-bottom-color: var(--border-color);
	padding-bottom:0.1em; /*step 2.2 task*/
	font-weight: bold; /*step 2.2 task*/
}

nav a:hover {
	border-bottom-width:0.4em; /*step 2.2 task extra*/
	border-bottom-style:solid; /*step 2.2 task extra*/
	border-bottom-color:var(--color-accent); /*step 2.2 task extra*/
	padding-bottom:0.1em; /*step 2.2 task extra*/
	background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
}

.color-scheme{
	position:absolute;
	top: 1rem;
	right: 1rem;
	display: inline-flex;
	gap: 5px;
}
</style>