<script lang="ts">
    import logo from "$lib/assets/logo.svg";

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

<svelte:head>
    <title>Longhorn Hacks — Computer science education for every community</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
    <link href="https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@75..125,400..800&family=Space+Grotesk:wght@500;600;700&family=Manrope:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
</svelte:head>

<main class="relative">
    <section class="relative overflow-hidden bg-lhh-ink py-[clamp(16px,5vh,32px)] px-[clamp(18px,3.4vw,40px)] min-h-screen flex items-center">
        <!-- Decorative circles (simplified) -->
        <div aria-hidden="true" class="absolute right-[-16%] bottom-[-38%] w-[min(900px,88vw)] aspect-square pointer-events-none rounded-full border border-lhh-orange-soft/30 grid place-items-center">
            <div class="w-[74%] aspect-square rounded-full border border-lhh-orange-soft/20 grid place-items-center">
                <div class="w-[62%] aspect-square rounded-full border border-lhh-orange-soft/16 border-dashed"></div>
            </div>
        </div>
        <!-- Bottom border -->
        <div aria-hidden="true" class="absolute left-0 right-0 bottom-0 h-px bg-lhh-cream/12 pointer-events-none"></div>
        <!-- Dot pattern background -->
        <div class="absolute inset-0 pointer-events-none opacity-50 bg-[radial-gradient(rgba(247,244,239,.16)_1px,transparent_1px)] bg-size-[26px_26px] mask-[radial-gradient(70%_70%_at_40%_40%,#000,transparent_76%)]"></div>

        <div class="relative z-10 max-w-335 mx-auto w-full -mt-15">
            <div class="grid grid-cols-1 md:grid-cols-[minmax(0,1.12fr)_minmax(0,.88fr)] gap-[clamp(28px,4vw,56px)] items-center">
                <div>
                    <div class="inline-flex items-center gap-2.5 px-3.5 py-1.5 rounded-full border border-lhh-orange-soft/30 bg-lhh-orange/10 font-mono text-[10.5px] tracking-[.18em] text-lhh-orange-soft">
                        <span class="w-1.5 h-1.5 rounded-full bg-lhh-orange shadow-[0_0_12px_#FF6A00] animate-pulse"></span>AUSTIN, TEXAS — NONPROFIT
                    </div>
                    <h1 class="mt-[clamp(20px,2.6vw,32px)] mb-0 font-display font-bold text-[clamp(38px,6.6vw,96px)] leading-[.9] tracking-[-.02em] uppercase">
                        <span class="block text-lhh-cream">Bridging the</span>
                        <span class="block text-transparent [-webkit-text-stroke:1.6px_var(--color-lhh-orange)]">Digital Divide</span>
                    </h1>
                    <p class="mt-[clamp(20px,2.4vw,30px)] mb-0 max-w-[44ch] text-[clamp(15.5px,1.3vw,18px)] leading-[1.62] text-lhh-blue-gray">Free computer science and AI education — plus the hardware to use it — for communities across Texas the tech economy left out. One line of code at a time.</p>
                    <div class="flex flex-wrap gap-3 mt-[clamp(26px,3vw,38px)]">
                        <a href="/contact" class="inline-flex items-center gap-3 px-5 py-2 rounded-full bg-lhh-orange font-mono text-[11.5px] tracking-[.14em] uppercase font-medium shadow-[0_8px_26px_rgba(255,106,0,.4)] hover:bg-lhh-orange-soft hover:shadow-[0_12px_36px_rgba(255,138,43,.6)] transition-all duration-300 ease-in-out"><p class="text-lhh-ink">Get involved</p><span class="grid place-items-center w-8 h-8 rounded-full bg-lhh-ink text-lhh-orange-soft text-lg">→</span></a>
                        <a href="/programs" class="inline-flex items-center gap-3 px-5 py-2 rounded-full border border-lhh-cream/20 text-lhh-cream font-mono text-[11.5px] tracking-[.14em] uppercase hover:border-lhh-orange-soft hover:text-lhh-orange-soft transition-all duration-300 ease-in-out">Explore programs<span class="grid place-items-center w-8 h-8 rounded-full bg-lhh-cream/8 text-lg">→</span></a>
                    </div>
                </div>

                <div class="relative grid place-items-center min-h-[min(58vw,440px)]">
                    <!-- Simplified SVG circles (no animation) -->
                    <svg viewBox="0 0 400 400" aria-hidden="true" class="absolute w-[min(430px,100%)] overflow-visible">
                        <circle cx="200" cy="200" r="188" fill="none" stroke="#F7F4EF" stroke-opacity=".12" stroke-width="1" stroke-dasharray="4 8" />
                        <circle cx="200" cy="200" r="152" fill="none" stroke="#FF6A00" stroke-opacity=".3" stroke-width="1" />
                        <g fill="none" stroke="#FF6A00" stroke-width="1.2">
                            <path d="M4 200 H58 V246 H100" />
                            <path d="M396 200 H342 V154 H300" />
                        </g>
                        <circle cx="4" cy="200" r="4" fill="#FF6A00" />
                        <circle cx="396" cy="200" r="4" fill="#6E90E8" />
                    </svg>
                    <div class="relative grid place-items-center w-[min(300px,74%)] aspect-square rounded-full bg-lhh-cream shadow-[0_0_0_1px_rgba(255,138,43,.4),0_0_110px_rgba(255,106,0,.55),0_34px_80px_rgba(0,0,0,.55)]">
                        <img src={logo} alt="Longhorn Hacks circuit longhorn logo" class="w-[76%] h-auto" />
                    </div>

                    <!-- Stat cards -->
                    <!-- Gonna comment out these stat cards until we have stats to put in them
                    <div class="absolute top-[4%] left-[-6%] flex items-center gap-3 px-4 py-2.5 rounded-xl border border-lhh-cream/14 bg-lhh-ink/80 backdrop-blur-md shadow-[0_16px_40px_rgba(0,0,0,.45)]">
                        <span class="grid place-items-center w-[30px] h-[30px] rounded-lg bg-lhh-orange/16 text-lhh-orange-soft font-mono text-xs">✳</span>
                        <span class="flex flex-col gap-0.5"><span class="font-mono text-[9.5px] tracking-[.16em] text-lhh-blue-gray-light">STUDENTS REACHED</span><span class="font-mono font-medium text-xs text-lhh-orange-soft">&lt;STATS GO HERE&gt;</span></span>
                    </div>
                    <div class="absolute bottom-[12%] right-[-8%] flex items-center gap-3 px-4 py-2.5 rounded-xl border border-lhh-cream/14 bg-lhh-ink/80 backdrop-blur-md shadow-[0_16px_40px_rgba(0,0,0,.45)]">
                        <span class="grid place-items-center w-[30px] h-[30px] rounded-lg bg-lhh-blue/16 text-lhh-blue-soft font-mono text-xs">◈</span>
                        <span class="flex flex-col gap-0.5"><span class="font-mono text-[9.5px] tracking-[.16em] text-lhh-blue-gray-light">DEVICES DONATED</span><span class="font-mono font-medium text-xs text-lhh-blue-soft">&lt;STATS GO HERE&gt;</span></span>
                    </div>
                    -->
                    <div class="absolute bottom-[-2%] left-[2%] -rotate-6 px-3.5 py-2 rounded-full bg-lhh-cream text-lhh-ink border-2 border-lhh-ink font-mono text-[10.5px] tracking-[.14em] shadow-[0_10px_26px_rgba(0,0,0,.4)]">FREE — ALWAYS</div>
                </div>
            </div>

            <div class="flex items-center justify-between gap-5 mt-[clamp(34px,4.4vw,60px)]">
                <span class="font-mono text-[10.5px] tracking-[.18em] text-lhh-blue-gray-darker text-right">EST. AUSTIN / TX</span>
            </div>
        </div>
    </section>

    <div class="relative overflow-hidden bg-lhh-orange text-lhh-ink py-3.5 border-t border-lhh-ink/20 border-b border-lhh-ink/20">
        <div class="flex gap-0 w-max animate-marquee">
            <div class="flex items-center gap-6 pr-6 font-display font-bold text-base tracking-[.04em] uppercase whitespace-nowrap">
                <span>Free CS &amp; AI lessons</span><span>✳</span><span>Device donations</span><span>✳</span><span>Hackathons</span><span>✳</span><span>Summer programs</span><span>✳</span><span>Nonprofit partnerships</span><span>✳</span>
            </div>
            <div aria-hidden="true" class="flex items-center gap-6 pr-6 font-display font-bold text-base tracking-[.04em] uppercase whitespace-nowrap">
                <span>Free CS &amp; AI lessons</span><span>✳</span><span>Device donations</span><span>✳</span><span>Hackathons</span><span>✳</span><span>Summer programs</span><span>✳</span><span>Nonprofit partnerships</span><span>✳</span>
            </div>
        </div>
    </div>

    <p class="mt-20 text-4xl text-center">There's never been a better time to learn to code.</p>
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

    <div class="typing-section">
        <p class="typing-line font-mono">
            Raising tomorrow's<br>
            <span class="typed-word">{typedText}</span><span class="cursor" aria-hidden="true"></span>
        </p>
    </div>
</main>

<style>
    /* Custom utility classes for colors and animations */
    .bg-lhh-ink { background-color: #04091A; }
    .text-lhh-ink { color: #04091A; }
    .bg-lhh-cream { background-color: #F7F4EF; }
    .text-lhh-cream { color: #F7F4EF; }
    .bg-lhh-orange { background-color: #FF6A00; }
    .text-lhh-orange-soft { color: #FF8A2B; }
    .hover\:text-lhh-orange-soft:hover { color: #FF8A2B; }
    .hover\:bg-lhh-orange-soft:hover { background-color: #FF8A2B; }
    .text-lhh-blue-gray { color: #B9C0D4; }
    .text-lhh-blue-gray-darker { color: #5E688A; }

    .font-display { font-family: 'Archivo', sans-serif; }
    .font-mono { font-family: 'JetBrains Mono', monospace; }

    @keyframes marquee {
        from { transform: translateX(0); }
        to { transform: translateX(-50%); }
    }
    .animate-marquee {
        animation: marquee 34s linear infinite;
    }

    @keyframes pulse {
        0%,100% { opacity: .25; }
        50% { opacity: 1; }
    }
    .animate-pulse {
        animation: pulse 2.2s ease-in-out infinite;
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
