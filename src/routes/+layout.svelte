<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
    import logo_blue_bg from '$lib/assets/logo-blue-bg.svg';
    import logo_blazing_orange from '$lib/assets/logo-blazing-orange.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<nav class="md:h-15 h-10 flex glass-nav fixed
 text-bright-orange font-mono border-round self-center
 rounded-xl items-center top-3 z-50 shadow">
    <div class="mx-0 inline-block">
        <a href="/" class="align-top"><img class="md:h-14.5 h-9.5" src={logo_blazing_orange} alt="Logo"></a>
    </div>
    <div class="ml-auto mx-8 align-middle justify-right">
        <a href="/" class="md:mx-4 mx-1.5 no-underline md:text-xl text-xs">Home</a>
        <a href="/about" class="md:mx-4 mx-1.5 no-underline md:text-xl text-xs">About</a>
        <a href="/contact" class="md:mx-4 mx-1.5 no-underline md:text-xl text-xs">Contact</a>
        <a href="/programs" class="md:mx-4 mx-1.5 no-underline md:text-xl text-xs">Programs</a>
        <a href="/events" class="md:mx-4 mx-1.5 no-underline md:text-xl text-xs">Events</a>
    </div>
</nav>
<div class="md:pt-21 pt-16 flex flex-col grow overscroll-none">
    <div class="grow text-text-grey
    font-extralight font-body">
        {@render children()}
    </div>
    <footer class="flex text-burnt-orange bg-dark-navy-blue border-t-2
    border-blazing-orange bottom-0 w-screen h-16 items-center">
        <div class="flex grow items-center justify-items-start justify-start">
            <a href="/" class=""><img class="h-12.5" src={logo_blue_bg} alt="Logo"></a>
            <a href="/legal" class="mx-4 no-underline text-lg align-middle font-mono">Legal</a>
        </div>
        <p class="mx-12 text-sm align-middle font-mono">
            This website is Open Source:
            <a href="https://github.com/longhornhacks/website" target="_blank" rel="noopener noreferrer">Source Code</a>
        </p>
    </footer>
</div>