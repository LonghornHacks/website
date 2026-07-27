<svelte:head><title>Longhorn Hacks - Home</title></svelte:head>

<script lang="ts">
    import logo from "$lib/assets/logo.svg";

    // Tracks how far the user has scrolled through the reveal section.
    let scrollY = $state(0);

    // The distance (in px) over which the effect fully plays out.
    const revealDistance = 600;

    // 0 at the top, clamped to 1 once fully scrolled through the reveal.
    let progress = $derived(Math.min(scrollY / revealDistance, 1));

    // Logo scrolls up slower than the page (parallax) and blurs/fades out.
    let logoTranslate = $derived(-scrollY * 0.35);
    let logoBlur = $derived(progress * 16);
    let logoOpacity = $derived(1 - progress);

    // Body text fades and slides into view as the logo dissolves.
    let bodyOpacity = $derived(progress);
    let bodyTranslate = $derived((1 - progress) * 40);

    // Typewriter effect: cycles through words after the static prefix.
    const words = ["Leaders", "Problem Solvers", "Innovators", "Builders", "Hackers"];
    const typingSpeed = 90;   // ms per character while typing
    const deletingSpeed = 45; // ms per character while deleting
    const pauseAfterWord = 1300; // ms to hold a fully-typed word
    const pauseAfterDelete = 300; // ms to hold before typing the next word

    let typedText = $state("");

    $effect(() => {
        let wordIndex = 0;
        let charIndex = 0;
        let deleting = false;
        let timer: ReturnType<typeof setTimeout>;

        const tick = () => {
            const current = words[wordIndex];

            if (!deleting) {
                charIndex++;
                typedText = current.slice(0, charIndex);

                if (charIndex === current.length) {
                    deleting = true;
                    timer = setTimeout(tick, pauseAfterWord);
                    return;
                }

                timer = setTimeout(tick, typingSpeed);
            } else {
                charIndex--;
                typedText = current.slice(0, charIndex);

                if (charIndex === 0) {
                    deleting = false;
                    wordIndex = (wordIndex + 1) % words.length;
                    timer = setTimeout(tick, pauseAfterDelete);
                    return;
                }

                timer = setTimeout(tick, deletingSpeed);
            }
        };

        timer = setTimeout(tick, typingSpeed);

        return () => clearTimeout(timer);
    });
</script>

<svelte:window bind:scrollY />

<div class="reveal-hero">
    <div
        class="hero-content flex flex-col-reverse"
        style="
            transform: translateY({logoTranslate}px);
            filter: blur({logoBlur}px);
            opacity: {logoOpacity};
        "
    >
        <img
            src={logo}
            alt="Longhorn Hacks logo"
            class="block mx-auto md:-mt-30 -mt-10 w-full max-w-3xl h-auto"
        />
        <p class="text-6xl text-center font-mono">Longhorn Hacks</p>
    </div>
</div>

<div
    class="reveal-body mx-auto max-w-3xl px-6"
    style="opacity: {bodyOpacity}; transform: translateY({bodyTranslate}px);"
>
    <p class="text-4xl text-center">There's never been a better time to learn to code.</p>
    <br>
    <hr>
    <br>
    <p>
        With AI technology revolutionizing the field and new projects starting every day,
        a new frontier has opened where anyone with the skills to code and the imagination to innovate
        could change the world.
    </p>
    <br>
    <p class="text-center text-3xl">This is your sign to get started.</p>
    <br>
    <p>
        Longhorn Hacks is on a mission to spread AI and Computer Science literacy across Central Texas.
        Additionally, we're committed to leveling the playing field, targeting the
        underprivileged communities that need our help the most.
    </p>
    <br>
    <p>
        Head over the the <a href="/events">events page</a> to see when we're visiting your
        community next.
    </p>
</div>

<div class="typing-section">
    <p class="typing-line font-mono">
        Raising tomorrow's<br>
        <span class="typed-word">{typedText}</span><span class="cursor" aria-hidden="true"></span>
    </p>
</div>

<style>
    .reveal-hero {
        position: sticky;
        top: 0;
        display: flex;
        flex-direction: column;
        justify-content: center;
        min-height: 100vh;
        z-index: 0;
    }

    .hero-content {
        will-change: transform, filter, opacity;
    }

    .reveal-body {
        position: relative;
        z-index: 1;
        will-change: transform, opacity;
    }

    .typing-section {
        position: relative;
        z-index: 1;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: clamp(5dvh, 4rem, 15dvh) 1.5rem;
        text-align: center;
    }

    .typing-line {
        font-size: clamp(2rem, 6vw, 4.5rem);
        font-weight: 700;
        line-height: 1.15;
    }

    .typed-word {
        color: var(--color-blazing-orange);
    }

    .cursor {
        display: inline-block;
        width: 0.08em;
        height: 1em;
        margin-left: 0.05em;
        background-color: var(--color-blazing-orange);
        vertical-align: text-bottom;
        animation: cursor-blink 1s steps(1) infinite;
    }

    @keyframes cursor-blink {
        0%, 50% {
            opacity: 1;
        }
        50.01%, 100% {
            opacity: 0;
        }
    }
</style>
