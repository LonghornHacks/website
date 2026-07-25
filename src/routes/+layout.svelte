<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
    import logo_blue_bg from '$lib/assets/logo-blue-bg.svg';
    import logo_blazing_orange from '$lib/assets/logo-blazing-orange.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<nav class="h-15 flex w-screen border-b-burnt-orange border-b-2
 bg-dark-navy-blue text-bright-orange font-sans border-round
 rounded-b-xl items-center">
    <div class="mx-0 inline-block">
        <a href="/" class="align-top"><img class="h-14.5" src={logo_blazing_orange} alt="Longhorn Hacks White Logo"></a>
    </div>
    <div class="ml-auto mx-8 align-middle justify-right">
        <a href="/" class="mx-4 no-underline text-xl">Home</a>
        <a href="/about" class="mx-4 no-underline text-xl">About</a>
        <a href="/contact" class="mx-4 no-underline text-xl">Contact</a>
        <a href="/programs" class="mx-4 no-underline text-xl">Programs</a>
        <a href="/events" class="mx-4 no-underline text-xl">Events</a>
    </div>
</nav>
<div class="grow bg-background-grey text-text-grey font-extralight font-sans">
    {@render children()}
</div>
<footer class="flex text-burnt-orange bg-dark-navy-blue border-t-2 border-blazing-orange bottom-0 w-screen h-16 items-center">
    <div class="flex grow items-center justify-items-start justify-start">
        <a href="/" class=""><img class="h-12.5" src={logo_blue_bg} alt="Longhorn Hacks Logo"></a>
        <a href="/legal" class="mx-4 no-underline text-lg align-middle">Legal</a>
    </div>
    <p class="mx-12 text-sm align-middle">
        This website is Open Source:
        <a href="https://github.com/longhornhacks/website" target="_blank" rel="noopener noreferrer">Source Code</a>
    </p>
</footer>
