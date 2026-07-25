<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<nav class="h-15 flex w-screen border-b-burnt-orange border-b-2
 bg-dark-navy-blue text-bright-orange font-sans border-round
 rounded-b-xl items-center">
    <div class="mx-0 flex-2">
        <a href="/" class="h-full"><img class="h-14.5 bg-bright-blue rounded-xl" src={favicon} alt="Longhorn Hacks Logo"></a>
    </div>
    <div class="flex-1 mr-auto mx-8 align-middle">
        <a href="/about" class="mx-4 no-underline">About</a>
        <a href="/contact" class="mx-4">Contact</a>
        <a href="/programs" class="mx-4">Programs</a>
        <a href="/events" class="mx-4">Events</a>
    </div>
</nav>
<div class="bg-dark-navy-blue text-blazing-orange font-extralight font-sans h-screen">
    {@render children()}
</div>