<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
    import logo_blue_bg from '$lib/assets/logo-blue-bg.svg';
    import logo_blazing_orange from '$lib/assets/logo-blazing-orange.svg';
    import extern_link from '$lib/assets/external-link-bright-orange.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<nav class="md:h-15 h-10 flex glass-nav fixed
 text-bright-orange font-mono self-center
  items-center top-3 z-50 shadow">
    <div class="mx-0 inline-block">
        <a href="/" class="align-top"><img class="md:h-14.5 h-9.5" src={logo_blazing_orange} alt="Logo"></a>
    </div>
    <div class="ml-auto w-fit md:mr-8 mr-4 align-middle justify-right">
        <a href="/" class="md:mx-4 mx-1.5 no-underline md:text-xl text-[11px]">Home</a>
        <a href="/about" class="md:mx-4 mx-1.5 no-underline md:text-xl text-[11px]">About</a>
        <a href="/programs" class="md:mx-4 mx-1.5 no-underline md:text-xl text-[11px]">Programs</a>
        <a href="/events" class="md:mx-4 mx-1.5 no-underline md:text-xl text-[11px]">Events</a>
        <a href="" class="md:mx-4 mx-1.5 no-underline md:text-xl text-[11px]"> <!--target="_blank" rel="noopener noreferrer"-->
            Donate<img src={extern_link} alt="External Link" class="external-link-nav"/>
        </a>
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
            <div class="flex md:flex-row flex-col">
                <a href="/legal" class="mx-4 no-underline md:text-lg text-sm align-middle font-mono">Legal</a>
                <a href="/contact" class="mx-4 no-underline md:text-lg text-sm align-middle font-mono">Contact</a>
            </div>
        </div>
        <p class="md:mx-12 mx-8 md:ml-12 ml-2 md:text-sm text-xs align-middle font-mono">
            This website is <a href="https://github.com/longhornhacks/website" target="_blank" rel="noopener noreferrer">
            Open Source<img src={extern_link} alt="External Link" class="external-link-footer"/></a>
        </p>
    </footer>
</div>

<style>
    .external-link-nav {
        display: inline-block;
        height: 12px;
        width: 12px;

        @media (width >= 48rem) {
            height: 22px;
            width: 22px;
        }
    }

    .external-link-footer {
        display: inline-block;
        height: 13px;
        width: 13px;

        @media (width >= 48rem) {
            height: 16px;
            width: 16px;
        }
    }
</style>