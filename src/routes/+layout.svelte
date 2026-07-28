<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
    import logo from '$lib/assets/logo.svg';
    import extern_link from '$lib/assets/external-link-bright-orange.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<nav class="md:h-15 h-10 flex glass-nav fixed self-center
  items-center top-3 z-50 md:pl-4 pl-1 md:pr-2.5 pr-1 gap-1">
    <a href="/" class="no-underline flex items-center gap-2.5 align-middle">
        <span class="grid place-items-center md:w-9.5 md:h-9.5 w-7 h-7 rounded-full bg-lhh-cream">
            <img class="md:w-9.5 md:h-9.5 w-7 h-7 object-contain" src={logo} alt="Logo">
        </span>
        <span class="hidden sm:inline font-display font-bold uppercase tracking-wide text-lhh-cream md:text-sm text-[0.65rem] whitespace-nowrap">Longhorn&nbsp;Hacks</span>
    </a>
    <div class="ml-auto flex items-center md:gap-1 gap-0.5 md:p-1 p-0.5 rounded-full
     bg-[rgba(247,244,239,0.045)] border border-[rgba(247,244,239,0.07)]">
        <a href="/" class="lhh-nav-link no-underline rounded-full md:px-4 px-0.5 py-2 md:text-[11px] text-[7px]">Home</a>
        <a href="/about" class="lhh-nav-link no-underline rounded-full md:px-4 px-0.5 py-2 md:text-[11px] text-[7px]">About</a>
        <a href="/programs" class="lhh-nav-link no-underline rounded-full md:px-4 px-0.5 py-2 md:text-[11px] text-[7px]">Programs</a>
        <a href="/events" class="lhh-nav-link no-underline rounded-full md:px-4 px-0.5 py-2 md:text-[11px] text-[7px]">Events</a>
    </div>
    <a href="" class="lhh-cta no-underline inline-flex items-center gap-1 md:px-5 px-1 py-2 md:text-[11px] text-[7px] ml-0">
        Donate<img src={extern_link} alt="External Link" class="external-link-nav"/>
    </a>
</nav>

<div class="md:pt-24 pt-20 flex flex-col grow overscroll-none">
    <div class="grow text-text-grey font-extralight font-body mx-[clamp(16px,8vw,64px)] max-w-5xl px-6 self-center">
        {@render children()}
    </div>
    <footer class="relative overflow-hidden bg-lhh-ink border-t border-[rgba(247,244,239,0.1)]
     text-lhh-muted md:px-10 px-5 md:pt-16 pt-12 pb-7 w-full">
        <div class="max-w-335 mx-auto">
            <div class="grid gap-x-14 gap-y-10 items-start"
             style="grid-template-columns:repeat(auto-fit,minmax(200px,1fr))">
                <div>
                    <div class="flex items-center gap-3">
                        <span class="grid place-items-center w-11.5 h-11.5 rounded-full bg-lhh-cream">
                            <img class="w-11 h-11 object-contain" src={logo} alt="Longhorn Hacks logo">
                        </span>
                        <span class="font-display font-bold uppercase tracking-wide text-lhh-cream text-base">Longhorn Hacks</span>
                    </div>
                    <p class="mt-4 text-lhh-dim text-sm leading-relaxed max-w-[30ch]">
                        Expanding access to computer science and AI education across Texas.
                    </p>
                </div>
                <div class="flex flex-col gap-3">
                    <span class="lhh-foot-label text-[9.5px] mb-1">Site</span>
                    <a href="/" class="lhh-foot-link no-underline text-sm">Home</a>
                    <a href="/about" class="lhh-foot-link no-underline text-sm">About</a>
                    <a href="/programs" class="lhh-foot-link no-underline text-sm">Programs</a>
                </div>
                <div class="flex flex-col gap-3">
                    <span class="lhh-foot-label text-[9.5px] mb-1">Get Involved</span>
                    <a href="/events" class="lhh-foot-link no-underline text-sm">Events</a>
                    <a href="/contact" class="lhh-foot-link no-underline text-sm">Contact</a>
                    <a href="" class="lhh-foot-link no-underline text-sm">Donate</a>
                </div>
                <div class="flex flex-col gap-3">
                    <span class="lhh-foot-label text-[9.5px] mb-1">Legal</span>
                    <a href="/legal" class="lhh-foot-link no-underline text-sm">Legal &amp; Privacy</a>
                    <p class="text-lhh-dim text-[13px] leading-relaxed max-w-[34ch]">
                        Longhorn Hacks is a registered 501(c)(3) nonprofit organization.
                    </p>
                </div>
            </div>
            <div class="mt-8 pt-4.5 border-t border-[rgba(247,244,239,0.1)] flex flex-wrap gap-3.5 items-center justify-between">
                <span class="font-mono text-[10.5px] tracking-wider text-[#5E688A]">© 2026 Longhorn Hacks</span>
                <p class="font-mono text-[10.5px] tracking-wider text-lhh-dim">
                    This website is <a href="https://github.com/longhornhacks/website" target="_blank" rel="noopener noreferrer" class="text-lhh-orange-soft">
                    Open Source<img src={extern_link} alt="External Link" class="external-link-footer"/></a>
                </p>
            </div>
        </div>
    </footer>
</div>

<style>
    .external-link-nav {
        display: inline-block;
        height: 10px;
        width: 10px;

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