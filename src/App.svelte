<script>
    import { onMount } from "svelte";
    import { fade } from "svelte/transition";

    import { gsap } from "gsap";
    import { ScrollSmoother } from "gsap/ScrollSmoother";
    import { ScrollToPlugin } from "gsap/ScrollToPlugin";
    import { ScrollTrigger } from "gsap/ScrollTrigger";
    import { Mouse, ChevronDown, Hammer, Lightbulb, Trophy, Truck } from "lucide-svelte";
    import p5 from "p5";

    import hackclub from "./assets/hackclub.svg";
    import parthenon from "./assets/events/parthenon.webp";
    import shipwrecked from "./assets/events/shipwrecked.webp";
    import undercity from "./assets/events/undercity.webp";
    import mascotDark from "./assets/mascot_dark.svg";
    import orpheus from "./assets/orpheus.svg";
    import Event from "./lib/Event.svelte";
    import Question from "./lib/Question.svelte";

    /** @type {HTMLElement | undefined} */
    let overlay;
    /** @type {HTMLDivElement | undefined} */
    let horizontalScroller;
    /** @type {HTMLElement | undefined} */
    let horizontalSection;
    /** @type {gsap.core.Tween | undefined} */
    let horizontalScrollTween = $state(undefined);
    /** @type {HTMLElement | undefined} */
    let stepsSection;
    /** @type {HTMLDivElement | undefined} */
    let stepsVisual;
    /** @type {ScrollSmoother} */
    let smoothie;
    let carouselIndex = $state(0);
    let activeStepIndex = $state(0);

    // thanks, macondo
    // TODO: add hctg bc holy peak
    const events = [
        {
            image: parthenon,
            label: "150 girls in NYC, at Parthenon",
            link: "https://www.youtube.com/watch?v=7K_E7tG-O68",
        },
        {
            image: shipwrecked,
            label: "150 teens in Boston, at Shipwrecked",
            link: "https://shipwrecked.hackclub.com",
        },
        {
            image: undercity,
            label: "100 teens in San Francisco, at Undercity",
            link: "https://www.youtube.com/watch?v=kaEFv7e49mo",
        },
    ];

    let activeEvent = $derived(events[carouselIndex]);

    // TODO: change header font for timeline or figure out a diff way to represent
    // NOTE: looks shitty asf, i think it's bc of Unbounded header font -@technodot
    const timeline = [
        {
            left: "5dvw",
            placement: "top",
            date: "2018",
            title: "TABNINE RELEASED",
            description: "The first integration of generative AI in software development.",
        },
        {
            left: "20dvw",
            placement: "bottom",
            date: "AUG 2021",
            title: "GPT-3 CODEX RELEASED",
            description: "The very first Codex model was released; a code-specialized version of GPT-3.",
        },
        {
            left: "25dvw",
            placement: "top",
            date: "JUN 2021",
            title: "GITHUB COPILOT RELEASED",
            description: "Leveraging GPT-3 Codex, GitHub Copilot offered seamless integration of AI into coding.",
        },
        {
            left: "40dvw",
            placement: "bottom",
            date: "NOV 30, 2022",
            title: "CHATGPT RELEASED",
            description: "Drew global public attention towards Artifical Intelligence and its potential capabilities.",
        },
        {
            left: "50dvw",
            placement: "top",
            date: "MAR 2023",
            title: "CURSOR RELEASED",
            description: "The first replacement of traditional code editors with a more AI-oriented solution.",
        },
        {
            left: "60dvw",
            placement: "bottom",
            date: "MAR 2024",
            title: "DEVIN RELEASED",
            description: "The world's first fully autonomous AI software engineer, operating with minimal human intervention.",
        },
        {
            left: "80dvw",
            placement: "top",
            date: "FEB 2025",
            title: "VIBECODING TERMED",
            description: 'AI researcher Andrej Karpathy coins the term "vibe coding" where users just talk to AI and forget the actual code exists.',
        },
        {
            left: "80dvw",
            placement: "bottom",
            date: "FEB 2025",
            title: "AI SLOP BUILDERS",
            description: "Tools like Base44 that allow anybody to build AI-powered applications without understanding any code.",
        },
        {
            left: "100dvw",
            placement: "top",
            date: "MAR 2025",
            title: "AI SECURITY SLOP",
            description: "curl founder Daniel Stenberg bans AI-generated security reports and condemns the tide of AI slop being submitted.",
        },
        {
            left: "105dvw",
            placement: "bottom",
            date: "2025",
            title: "VULNNERABLE CODE",
            description: "A noticeably sharp uptick in security vulnerabilities reported in AI-generated code, some even actively exploited.",
        },
        {
            left: "120dvw",
            placement: "top",
            date: "DEC 2025",
            title: "KIRO BREAKS AWS",
            description: "Amazon's internal AI coding agent deleted part of AWS's production infra and rebuilt it, leading to a 13 hour outage.",
        },
        {
            left: "125dvw",
            placement: "bottom",
            date: "FEB 2026",
            title: "AI BUBBLE?",
            description: "Forecasted >$6T in 2026 global spending. While 88% of organizations experiment with AI, 81% do not report any meaningful gains.",
            href: "https://www.mckinsey.com/capabilities/people-and-organizational-performance/our-insights/the-state-of-organizations",
        },
        {
            left: "145dvw",
            placement: "top",
            date: "MAR 2026",
            title: "KIRO BREAKS AMAZON",
            description: "6.3 million orders were lost across North America.",
        },
        {
            left: "145dvw",
            placement: "bottom",
            date: "MAR 2026",
            title: "AI FRUIT LOVE ISLAND",
            description: "no.",
        },
    ];

    /** @param {number} index */
    const activateStep = (index) => {
        if (activeStepIndex === index) return;

        activeStepIndex = index;
        gsap.to("#step-progress", { yPercent: index * 100, duration: 0.2, ease: "power2.out", overwrite: true });
    };

    onMount(() => {
        const carouselTimer = window.setInterval(() => {
            carouselIndex = (carouselIndex + 1) % events.length;
        }, 5 * 1000);

        gsap.registerPlugin(ScrollTrigger, ScrollSmoother, ScrollToPlugin);
        smoothie = ScrollSmoother.create({
            smooth: 1,
            effects: true,
            normalizeScroll: true,
        });

        const scrollerEl = horizontalScroller;
        const scrollerSectionEl = horizontalSection;
        horizontalScrollTween =
            scrollerEl &&
            scrollerSectionEl &&
            gsap.to(scrollerEl, {
                x: () => -Math.max(0, scrollerEl.scrollWidth - scrollerSectionEl.clientWidth),
                ease: "none",
                scrollTrigger: {
                    trigger: scrollerSectionEl,
                    start: "top top",
                    end: () => `+=${Math.max(0, scrollerEl.scrollWidth - scrollerSectionEl.clientWidth)}`,
                    scrub: true,
                    pin: true,
                    anticipatePin: 1,
                    invalidateOnRefresh: true,
                },
            });

        const renderer = new p5((p) => {
            /** @typedef {{ x: number; y: number; vx: number; vy: number }} CloudPoint */
            const LINK_DISTANCE = 143.11; // +25% of 128 (nonlinear!)
            /** @type {CloudPoint[]} */
            const points = [];

            const getPointCount = () => Math.max(67, Math.round((p.width * p.height * 128) / (1920 * 1080)));

            const randomVelocity = () => {
                const speed = p.random(0.1, 0.3);
                return speed * (p.random() > 0.5 ? 1 : -1);
            };

            const resetPoints = () => {
                points.length = 0;
                for (let i = 0; i < getPointCount(); i++) {
                    points.push({
                        x: p.random(p.width),
                        y: p.random(p.height),
                        vx: randomVelocity(),
                        vy: randomVelocity(),
                    });
                }
            };

            p.setup = () => {
                if (!overlay) return;
                const canvas = p.createCanvas(window.innerWidth, window.innerHeight);
                canvas.parent(overlay);
                canvas.style("pointer-events", "none");
                p.pixelDensity(1);
                p.frameRate(60);
                resetPoints();
            };

            p.draw = () => {
                p.clear();

                for (let i = 0; i < points.length; i++) {
                    const point = points[i];
                    point.x += point.vx;
                    point.y += point.vy;

                    if (point.x <= 0 || point.x >= p.width) point.vx *= -1;
                    if (point.y <= 0 || point.y >= p.height) point.vy *= -1;
                }

                const maxDistance = LINK_DISTANCE * LINK_DISTANCE;
                for (let i = 0; i < points.length; i++) {
                    for (let j = i + 1; j < points.length; j++) {
                        const dx = points[i].x - points[j].x;
                        const dy = points[i].y - points[j].y;
                        const d2 = dx * dx + dy * dy;
                        if (d2 > maxDistance) continue;

                        const alpha = (1 - d2 / maxDistance) * 67;
                        p.stroke(110, 255, 210, alpha); // NOTE: #6effd2
                        p.strokeWeight(1);
                        p.line(points[i].x, points[i].y, points[j].x, points[j].y);
                    }
                }

                p.noStroke();
                p.fill(110, 255, 210, 115); // NOTE: #6effd2
                for (let i = 0; i < points.length; i++) {
                    p.circle(points[i].x, points[i].y, 3.2);
                }
            };

            p.windowResized = () => {
                p.resizeCanvas(window.innerWidth, window.innerHeight);
                resetPoints();
            };
        });

        /** @type {Element[]} */
        const steps = gsap.utils.toArray(".step-item");
        const updateActiveStep = () => {
            if (!steps.length) return;

            const focusY = window.innerHeight * 0.52; // eye position
            let closestIndex = 0;
            let closestDistance = Number.POSITIVE_INFINITY;

            // find closest
            steps.forEach((step, index) => {
                const rect = step.getBoundingClientRect();
                const distance = Math.abs(rect.top + rect.height / 2 - focusY);

                if (distance < closestDistance) {
                    closestDistance = distance;
                    closestIndex = index;
                }
            });

            activateStep(closestIndex);
        };

        const stepStateTrigger = stepsSection
            ? ScrollTrigger.create({
                  trigger: stepsSection,
                  start: "top bottom",
                  end: "bottom top",
                  onEnter: updateActiveStep,
                  onEnterBack: updateActiveStep,
                  onUpdate: updateActiveStep,
                  onRefresh: updateActiveStep,
                  onLeave: () => activateStep(steps.length - 1),
                  onLeaveBack: () => activateStep(0),
                  invalidateOnRefresh: true,
              })
            : undefined;

        updateActiveStep();

        const stepProgressPin = ScrollTrigger.create({
            trigger: "#steps-progress-wrapper",
            start: "top 12.5%",
            endTrigger: ".w-1\\/2.flex.flex-col",
            end: "bottom 75%",
            pin: true,
            pinSpacing: false,
            invalidateOnRefresh: true,
        });

        const stepsVisualPinTrigger =
            stepsSection &&
            stepsVisual &&
            ScrollTrigger.create({
                trigger: stepsSection,
                start: "top top",
                end: "bottom bottom",
                pin: stepsVisual,
                pinSpacing: false,
                anticipatePin: 1,
                invalidateOnRefresh: true,
            });

        return () => {
            window.clearInterval(carouselTimer);

            // disable p5
            renderer.remove();

            // FIRST disable tweens
            horizontalScrollTween?.scrollTrigger?.kill();
            horizontalScrollTween?.kill();
            horizontalScrollTween = undefined;

            stepStateTrigger?.kill();

            stepProgressPin?.kill();

            stepsVisualPinTrigger?.kill();

            // disable smoothscroll
            smoothie.kill();
        };
    });
</script>

<div id="smooth-wrapper" class="relative overflow-x-clip bg-[linear-gradient(150deg,#080E12,#0B1618)]">
    <!-- cool visuals overlay -->
    <div bind:this={overlay} class="fixed inset-0 w-dvw h-dvh pointer-events-none z-0"></div>

    <div id="smooth-content" class="relative z-10">
        <!-- hackclub icon -->
        <section class="absolute top-6 left-1/2 -translate-x-1/2 z-50 flex items-center gap-2">
            <h6 class="text-(--accent-border) ml-[26.13px] tracking-wide">a hack</h6>
            <a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="block">
                <img src={hackclub} class="cursor-pointer transition-all duration-300 ease-out hover:-translate-y-1 hover:scale-105 hover:drop-shadow-xl active:scale-95 active:-translate-y-1 active:duration-150" width="40" alt="Hack Club logo" />
            </a>
            <h6 class="text-(--accent-border) tracking-wide">club ysws</h6>
        </section>

        <!-- orpheus flag -->
        <section class="absolute top-0 left-6">
            <a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="block">
                <img src={orpheus} class="cursor-pointer" width="180" alt="Orpheus flag" />
            </a>
        </section>

        <!-- hackanomous presents -->
        <!-- <section class="absolute top-[50dvh] left-1/2 translate-x-[-23rem] translate-y-[-11rem] -rotate-[15deg] z-1 underline underline-offset-2 decoration-(--accent-border) z-20">
            <h6 class="font-content text-(--accent-border) tracking-wider">hackanomous presents</h6>
        </section> -->

        <!-- landing -->
        <section class="relative min-h-dvh flex justify-center items-center py-12 px-4">
            <div class="relative">
                <div class="w-fit border-2 border-dashed border-(--code-bg) rounded-2xl px-16 py-2 relative z-10 -ml-24 -rotate-5 bg-[linear-gradient(175deg,var(--bg)_0%,#0B161850_33%,#080E1280_100%)]">
                    <h3 class="font-mono font-extralight text-sm text-(--text-h) tracking-widest">COMING SOON</h3>
                </div>
                <div class="flex">
                    <div class="text-right">
                        <h1 class="font-heading font-regular text-7xl text-(--accent) mt-4">
                            hackanomous
                        </h1>
                        <h1 class="font-heading font-regular text-4xl text-(--text) mt-4">
                            BUILD <span class="font-semibold underline">AI</span>. GET <span class="font-semibold underline">PRIZES</span>!
                        </h1>
                    </div>
                    <img src={mascotDark} class="base -mt-4.5 ml-2" width="216" alt="anomaly, our mascot!" />
                </div>
                <div class="flex relative z-10 pl-36 pr-50">
                    <div class="relative flex-1 mr-4">
                        <input id="rsvp-input" class="peer w-full px-4 font-mono font-light border-2 border-solid border-(--accent) text-(--text-h) rounded-xl py-3 focus:outline-none focus:border-(--accent-hover) focus:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_6.7%,transparent)] bg-transparent placeholder:text-(--accent-border) placeholder-opacity-100" style="transition: box-shadow 0.35s ease-out, border-color 0.15s ease-out, background-color 0.15s ease-out;" type="text" placeholder="you@example.com" />
                        <label for="rsvp-input" class="absolute left-3 -top-2.5 px-1 font-mono text-xs font-semibold text-(--accent-hover) bg-[#0a1215] opacity-0 translate-y-2 pointer-events-none transition-all duration-100 peer-focus:opacity-100 peer-focus:translate-y-0 rounded-sm border-2">email</label>
                    </div>
                    <button onclick={() => window.open("https://hackanomous-rsvp.fillout.com/t/4oPTMjqFuaus", "_blank")} class="font-mono font-semibold border-2 border-solid border-(--accent) bg-(--accent) text-(--bg) rounded-xl px-10 py-3 cursor-pointer hover:bg-transparent hover:text-(--accent) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_6.7%,transparent)]" style="transition: box-shadow 0.35s ease-out, border-color 0.15s ease-out, background-color 0.15s ease-out, color 0.15s ease-out;">RSVP!</button>
                </div>
                <h6 class="font-mono font-light text-xs text-center text-(--text-h) tracking-widest mt-4">
                    ages 13-18 only. <span class="underline underline-offset-2">Sept 1</span> to <span class="underline underline-offset-2">Jan 1</span>.
                </h6>
                <div class="flex flex-col absolute bottom-14 right-4 -rotate-5 font-mono text-xs text-(--accent-border)">
                    <span class="text-right">with lots of love,</span>
                    <span class="text-right">jacob abby & techno &lt;3</span>
                </div>
            </div>
            
            <!-- scroll down -->
            <section class="absolute bottom-8 left-1/2 -translate-x-1/2 text-(--accent-border)">
                <div class="flex flex-row items-center gap-3 floater">
                    <Mouse size={24} />
                    <span class="font-content font-light text-base tracking-widest">SCROLL DOWN</span>
                </div>
            </section>
        </section>

        <!-- TODO: vertical step by step scroll -->
        <section bind:this={stepsSection} class="min-h-dvh flex flex-col items-center relative overflow-hidden bg-black/80 w-full" id="steps-section">
            <div class="absolute top-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--accent) to-transparent opacity-67 z-10"></div>

            <div class="w-full max-w-7xl mx-auto flex flex-row relative py-32 px-6 lg:px-24">
                
                <!-- left content scrolling -->
                <div class="w-1/2 flex flex-col relative pr-20">
                    
                    <!-- steps progress bar -->
                    <div class="absolute inset-y-0 left-0 w-1 z-20">
                        <div id="steps-progress-wrapper" class="w-full h-[50vh] min-h-[50dvh] bg-(--code-bg)">
                            <div id="step-progress" class="w-full h-1/4 bg-(--accent) shadow-[0_0_50px_color-mix(in_srgb,var(--accent)_50%,transparent)]" style="transform: translateY(0%);"></div>
                        </div>
                    </div>

                    <!-- TODO: figure out if we need to add 01 02 03 04 to each step? -->

                    <!-- steps -->
                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 0} class:opacity-30={activeStepIndex !== 0}>
                        <div class="absolute">
                            <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">1</h1>
                        </div>
                        <h2 class="font-mono font-bold text-4xl text-(--accent-hover) mb-4 tracking-wide">INSPIRE TO CREATE</h2>
                        <p class="font-content font-light text-base text-(--text) leading-relaxed tracking-wide">
                            Brainstorm an innovative and interesting idea! Explore practical scenarios in which AI could actually make a productive difference.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 1} class:opacity-30={activeStepIndex !== 1}>
                        <div class="absolute">
                            <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">2</h1>
                        </div>
                        <h2 class="font-mono font-bold text-4xl text-(--accent-hover) mb-4 tracking-wide">BUILD YOUR PROJECT</h2>
                        <p class="font-content font-light text-base text-(--text) leading-relaxed tracking-wide">
                            Bring your idea to life through code and hardware! If you're making a hardware project and can't cover parts, we've got your back with a funding grant.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 2} class:opacity-30={activeStepIndex !== 2}>
                        <div class="absolute">
                            <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">3</h1>
                        </div>
                        <h2 class="font-mono font-bold text-4xl text-(--accent-hover) mb-4 tracking-wide">SHIP TO THE WHOLE<br>UNIVERSE</h2>
                        <p class="font-content font-light text-base text-(--text) leading-relaxed tracking-wide">
                            Show off your finished project! Also, check out what other amazing projects by other teenage inventors (like yourself!) have been building and shipping.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 3} class:opacity-30={activeStepIndex !== 3}>
                        <div class="absolute">
                            <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">4</h1>
                        </div>
                        <h2 class="font-mono font-bold text-4xl text-(--accent-hover) mb-4 tracking-wide">CLAIM YOUR EPIC REWARDS</h2>
                        <p class="font-content font-light text-base text-(--text) leading-relaxed tracking-wide">
                            You did it! In exchange for helping take AI to the next level, receive Bolts to get DDR5 RAM, AI credits, or perhaps an RTX 5090!
                        </p>
                    </div>

                </div>

                <!-- right side -->
                <div class="w-1/2 relative h-full">
                    <div bind:this={stepsVisual} class="relative flex aspect-square w-full items-center justify-center overflow-hidden border-2 border-(--accent) bg-white/1">
                        {#key activeStepIndex}
                            <section transition:fade={{ duration: 200 }} class="absolute inset-0 flex flex-col items-center justify-center gap-3 px-8 text-center font-mono text-(--text-h) will-change-opacity">
                                {#if activeStepIndex === 0}
                                    <div class="relative h-[78%] w-[78%]">
                                        <img src={mascotDark} alt="anomaly having an idea" class="absolute bottom-[15%] left-[calc(50%+18px)] h-[58%] -translate-x-1/2 object-contain" />
                                        <Lightbulb class="absolute left-1/2 top-[15%] h-24 w-24 -translate-x-1/2 text-(--accent) drop-shadow-[0_0_52px_color-mix(in_srgb,var(--accent-hover)_70%,transparent)]" strokeWidth={1.8} />
                                    </div>
                                {:else if activeStepIndex === 1}
                                    <div class="relative h-[78%] w-[78%]">
                                        <Hammer class="absolute bottom-[48%] left-[calc(50%+120px)] h-24 w-24 -translate-x-1/2 text-(--accent) drop-shadow-[0_0_24px_color-mix(in_srgb,var(--accent-hover)_10%,transparent)]" strokeWidth={1.9} />
                                        <img src={mascotDark} alt="anomaly holding a hammer" class="absolute bottom-[23%] left-[calc(50%-24px)] h-[56%] -translate-x-1/2 object-contain" />
                                    </div>
                                {:else if activeStepIndex === 2}
                                    <div class="relative h-[68%] w-[84%]">
                                        <Truck class="absolute bottom-[22%] left-1/2 h-[52%] w-[86%] -translate-x-1/2 text-(--accent) drop-shadow-[0_0_52px_color-mix(in_srgb,var(--accent-hover)_20%,transparent)]" strokeWidth={1.8} />
                                        <img src={mascotDark} alt="anomaly logo on truck side" class="absolute bottom-[42%] left-[42.5%] h-[20%] -translate-x-1/2 object-contain" />
                                    </div>
                                {:else}
                                    <div class="relative h-[78%] w-[48%]">
                                        <img src={mascotDark} alt="anomaly with a reward" class="absolute bottom-[16%] left-[calc(50%-18px)] h-[58%] -translate-x-1/2 object-contain" />
                                        <Trophy class="absolute left-[calc(50%-10px)] top-[19%] h-24 w-24 -translate-x-1/2 text-(--accent) drop-shadow-[0_0_34px_color-mix(in_srgb,var(--accent-hover)_48%,transparent)]" strokeWidth={1.8} />
                                    </div>
                                {/if}
                            </section>
                        {/key}
                    </div>
                </div>

            </div>

            <div class="absolute bottom-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--accent) to-transparent opacity-67 z-10"></div>
        </section>

        <section class="relative min-h-dvh px-6 md:px-12 xl:px-24 py-24 pb-36 flex flex-col justify-center items-center">
            <div class="max-w-6xl mx-auto w-full">
                <h1 class="font-heading text-3xl mb-8 tracking-wide">what is <span class="font-mono font-bold text-4xl text-(--accent)">Hackanomous</span>?</h1>
                <p class="font-content leading-relaxed"><span class="font-mono font-bold text-(--accent)">Hackanomous</span> is a <span class="font-mono font-bold text-(--accent)">YSWS</span> program where you design and ship <span class="font-mono font-bold text-(--accent)">AI-driven</span> personal projects: hardware or software.<br>We walk you through building your own projects while exploring both effective and ineffective usecases of AI, and ship you free rewards!<br><br>We're even hosting a hackathon in <span class="font-mono font-bold text-(--accent)">Islambad, Pakistan</span> to conclude the event! Qualify by earning enough hours to score an invite!</p>

                <div class="mt-14 w-full">
                    <div class="group relative block w-full origin-center transform-gpu overflow-hidden rounded-2xl border-2 border-(--code-bg) bg-black/70 aspect-16/10 sm:aspect-video lg:aspect-21/9 focus-within:border-(--accent) focus-within:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_5%,transparent)] hover:border-(--accent) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_4%,transparent)] motion-safe:hover:scale-[1.018] motion-safe:hover:rotate-[0.8deg] transition-all duration-500 ease-out will-change-transform">
                        <a href={activeEvent.link} target="_blank" rel="noopener noreferrer" aria-label={`Open ${activeEvent.label}`} class="absolute inset-0 z-10 cursor-pointer focus:outline-none"></a>
                        {#key carouselIndex}
                            <img transition:fade={{ duration: 300 }} src={activeEvent.image} alt={activeEvent.label} class="absolute inset-0 h-full w-full object-cover transition-transform duration-700 ease-out group-hover:scale-[1.03]" />
                        {/key}
                        <div class="absolute inset-0 bg-[linear-gradient(180deg,rgba(8,14,18,0)_28%,rgba(8,14,18,0.88)_100%)]"></div>
                        <div class="absolute inset-0 border border-white/10 rounded-[14px] pointer-events-none"></div>

                        <div class="pointer-events-none absolute left-4 right-4 bottom-4 z-20 md:left-6 md:right-6 md:bottom-6 flex flex-col gap-4 md:flex-row md:items-end md:justify-between">
                            <div>
                                <span class="font-content font-light text-sm tracking-widest">previous events!</span>
                                <h2 class="font-mono font-medium text-xl md:text-3xl text-(--text) leading-tight">{activeEvent.label}</h2>
                            </div>
                            <div class="pointer-events-auto flex gap-2" role="group" aria-label="Event carousel controls">
                                {#each events as event, index (event.label)}
                                    <button type="button" onclick={() => (carouselIndex = index)} aria-label={`Show ${event.label}`} aria-pressed={index === carouselIndex} class="flex h-5 w-10 items-center rounded-full border-none bg-transparent p-0 opacity-80 transition-opacity duration-300 hover:opacity-100 focus:outline-none focus-visible:shadow-[0_0_0_2px_var(--accent)] cursor-pointer">
                                        <span class={`h-1.5 w-full rounded-full transition-all duration-300 ${index === carouselIndex ? "bg-(--accent)" : "bg-(--accent-border)/50"}`}></span>
                                    </button>
                                {/each}
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ...why does this exist? -->
            <div class="absolute bottom-8 left-1/2 -translate-x-1/2 floater flex flex-col items-center">
                <span class="font-content font-light text-base tracking-widest">...why does this exist?</span>
                <ChevronDown size={24}/>
            </div>
        </section>

        <!-- timeline -->
        <section bind:this={horizontalSection} class="min-h-dvh flex flex-col justify-center items-center relative overflow-hidden bg-black/80">
            <!-- top border strip glow -->
            <div class="absolute top-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--accent) to-transparent opacity-67"></div>

            <div class="w-full overflow-hidden flex-1 relative">
                <div id="horizontal-scroller" bind:this={horizontalScroller} class="w-[200dvw] max-w-none flex items-center h-full relative">
                    <!-- base timeline line -->
                    <div class="absolute left-0 right-0 top-1/2 -translate-y-1/2 block h-0.5 w-[calc(150dvw-8px)] bg-(--text-h)"></div>

                    <!-- timeline events -->
                    {#each timeline as event (event.title)}
                        <Event {...event} scrollTween={horizontalScrollTween} />
                    {/each}

                    <!-- closing node (i.e. present) -->
                    <div class="absolute top-1/2 left-[150dvw] z-10 hover:z-20 group">
                        <div class="absolute w-4 h-4 rounded-full border-2 border-(--accent) -translate-x-1/2 -translate-y-1/2 transition-transform"></div>
                    </div>

                    <!-- right content -->
                    <div class="absolute top-1/2 left-[165dvw] -translate-y-[calc(50%+28px)] w-[85dvw] sm:w-[70dvw] md:w-[60dvw] lg:w-[45vw] pr-12 lg:pr-24 z-10 flex flex-col items-end text-right">
                        <h2 class="font-mono font-medium text-4xl md:text-6xl text-(--text) leading-tight">
                            <span class="italic">DE</span>SLOP THE<br />
                            <span class="font-mono font-bold inline-block bg-clip-text text-transparent bg-[linear-gradient(90deg,var(--accent)_0%,color-mix(in_srgb,var(--accent)_67%,transparent)_100%)]">WORLD.</span>
                        </h2>

                        <p class="font-mono font-normal text-lg md:text-xl leading-relaxed text-(--text-h) mt-6 max-w-2xl">
                            the AI bubble might just be about to pop.<br />
                            <span class="font-bold text-(--text-l)">YOUR MISSION:</span> build projects incorporating AI that solve <u><span class="font-bold text-(--text-l)">real-world</span></u> problems.
                        </p>

                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-6 mt-10 w-full max-w-3xl">
                            <!-- TODO: instead of shifting up on hover, make it lighten up -->
                            <div class="border border-(--code-bg) p-6 rounded-2xl origin-center transform-gpu hover:bg-white/3 hover:border-(--accent) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_6.7%,transparent)] motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-500 ease-out will-change-transform group cursor-default">
                                <h4 class="font-heading text-3xl text-(--accent) mb-2 group-hover:drop-shadow-[0_0_16px_color-mix(in_srgb,var(--accent)_20%,transparent)] transition-all duration-500 text-right">software</h4>
                                <!-- TODO: elaborate? -->
                                <p class="font-content text-sm font-light text-(--text-h) group-hover:text-(--text) transition-colors duration-500">Build software that implements AI or ML to earn Bolts! Use them to buy Raspberry Pis, API credits, RAM & GPU grants, and more!</p>
                            </div>
                            <div class="border border-(--code-bg) p-6 rounded-2xl origin-center transform-gpu hover:bg-white/3 hover:border-(--accent) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--accent-hover)_6.7%,transparent)] motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-500 ease-out will-change-transform group cursor-default">
                                <h4 class="font-heading text-3xl text-(--accent) mb-2 group-hover:drop-shadow-[0_0_16px_color-mix(in_srgb,var(--accent)_20%,transparent)] transition-all duration-500 text-right">hardware</h4>
                                <!-- TODO: elaborate? -->
                                <p class="font-content text-sm font-light text-(--text-h) group-hover:text-(--text) transition-colors duration-500">Design hardware that implements AI or ML and receive funding to build it! Earn bolts for your physical work!</p>
                            </div>
                        </div>

                        <div class="flex gap-5 mt-8">
                            <button onclick={() => gsap.to(window, { duration: .3, scrollTo: { y: '#faq', autoKill: true }, ease: 'power2.inOut' })} class="font-mono font-semibold border-2 border-solid border-(--accent) text-(--accent) hover:bg-(--accent) hover:text-(--bg) rounded-xl px-12 py-4 cursor-pointer focus:outline-none hover:shadow-[0_0_30px_color-mix(in_srgb,var(--accent)_30%,transparent)] transition-all duration-150 tracking-wide text-lg"> LEARN MORE </button>
                            <button onclick={() => gsap.to(window, { duration: .7, scrollTo: { y: 0, autoKill: true }, ease: 'power2.inOut' })} class="font-mono font-semibold border-2 border-solid border-(--accent) text-(--accent) hover:bg-(--accent) hover:text-(--bg) rounded-xl px-12 py-4 cursor-pointer focus:outline-none hover:shadow-[0_0_30px_color-mix(in_srgb,var(--accent)_30%,transparent)] transition-all duration-150 tracking-wide text-lg"> REGISTER NOW </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- bottom border strip glow -->
            <div class="absolute bottom-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--accent) to-transparent opacity-67"></div>
        </section>

        <!-- standard FAQ and closing info -->
        <section id="faq" class="min-h-dvh px-6 md:px-12 xl:px-24 py-24 pb-55 flex flex-col justify-start items-center relative">
            <!-- pb-[220px] = calc(16px + 180px + 24px) -->
            <!-- faq -->
            <div class="max-w-7xl mx-auto w-full">
                <div class="relative w-full mb-12">
                    <h1 class="font-heading font-regular text-5xl md:text-6xl text-(--text) w-full relative z-10">FAQ</h1>
                    <!-- real text -->
                    <div class="absolute top-0 left-0 font-heading font-regular text-5xl md:text-6xl text-(--accent)" style="transform: translate(6px, 3px);">FAQ</div>
                    <!-- offset text -->
                </div>

                <!-- TODO: answer more flipping questions -->
                <!-- TODO: also make actually look nice -->

                <div class="grid grid-cols-1 md:grid-cols-2 gap-x-12 lg:gap-x-24 gap-y-0 w-full">
                    <!-- column 1 -->
                    <div class="flex flex-col">
                        <Question question="Who can participate?">
                            <!-- vetted answer -->
                            <p>Anyone between the ages of <span class="font-mono font-bold text-(--accent)">13</span> and <span class="font-mono font-bold text-(--accent)">18</span> (inclusive) can participate in Hackanomous!</p>
                        </Question>
                        <Question question="What projects can I build?">
                            <!-- vetted answer -->
                            <p>You can build any project incorporating <span class="font-mono font-bold text-(--accent)">AI</span> or <span class="font-mono font-bold text-(--accent)">ML</span>! We want to see your innovation shine, whether it's software, hardware, or something COMPLETELY new and unique.</p>
                        </Question>
                        <Question question="I don't know anything about AI/ML!">
                            <!-- vetted answer -->
                            <p><span class="font-mono font-bold text-(--accent)">No problem!</span> We have resources and mentors to help you learn. Plus, the best way to learn is by building, so dive in and start experimenting!</p>
                        </Question>
                        <Question question="What are the prizes?">
                            <!-- vetted answer -->
                            <p>We have a variety of exciting prizes for our participants! Shipped projects will receive <span class="font-mono font-bold text-(--accent)">Bolts</span>, which can be redeemed for Raspberry Pis, AI credits, RAM & GPU grants, and more. We might even send you an <span class="font-mono font-bold text-(--accent)">RTX 5090</span>!</p>
                        </Question>
                        <Question question="Is this legit? What's Hack Club?">
                            <!-- vetted answer -->
                            <p><a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="font-mono font-bold underline text-(--accent)">Hack Club</a> is the world's largest community of teenage makers, and a 501(c)(3) nonprofit. Hack Club is supported by donations from tech companies like GitHub and individuals like Michael Dell. Hack Club is fiscally transparent.</p>
                        </Question>
                    </div>

                    <!-- column 2 -->
                    <div class="flex flex-col">
                        <Question question="How do I register?">
                            <!-- vetted answer -->
                            <p>Registration for Hackanomous is simple! Just scroll up and click the <span class="font-mono font-bold text-(--accent)">RSVP</span> button to fill out the registration form.</p>
                        </Question>
                        <Question question="How many projects can I build?">
                            <!-- vetted answer -->
                            <p>You can build as many projects as you like, but we encourage you to focus on <span class="font-mono font-bold text-(--accent)">quality over quantity</span>. Your rewards <span class="font-mono font-bold text-(--accent)">scale exponentially</span> with time, so really focus on building something amazing!</p>
                        </Question>
                        <Question question="How do I fund hardware?">
                            <!-- vetted answer -->
                            <p>Don't worry! You can get up to <span class="font-mono font-bold text-(--accent)">$100</span> <span class="font-mono font-bold text-(--accent)">(TODO: TBD)</span> to buy parts to build your hardware project.</p>
                        </Question>
                        <Question question="Is this free?">
                            <!-- vetted answer -->
                            <p><span class="font-mono font-bold text-(--accent)">Yes!</span> Hackanomous is <span class="font-mono font-bold text-(--accent)">100%</span> free - all the prizes are donated by sponsors or paid for by us! <span class="font-mono text-sm text-(--accent)">*customs may occur outside the US!</span></p>
                        </Question>
                        <Question question="I have more questions...">
                            <!-- vetted answer -->
                            <p>Ask us in <span class="font-mono font-bold text-(--accent)">#hackanomous-help</span> in Slack!</p>
                            <!-- switch out when we get an hc email -->
                            <!-- <p>Ask us in <span class="font-mono font-bold text-(--accent)">#hackanomous-help</span> in Slack, or email us at <a href="mailto:hackanomous@hackclub.com" class="font-mono font-bold underline text-(--accent)">hackanomous@hackclub.com</a>!</p> -->
                        </Question>
                    </div>
                </div>
            </div>

            <!-- closing info -->
            <div class="bg-black/80 absolute bottom-4 left-4 px-1">
                <span class="font-mono text-xs text-(--text-h) block">
                    <span class="text-(--accent)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span><br />
                    <span class="text-(--accent)">|&nbsp;</span><span class="font-bold text-(--text)"># Built by the Hackanomous Team @ Hack Club</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>Hack Club is a 501(c)(3) nonprofit and network of 67k+ technical high schoolers. We believe you<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>learn best by building so we're creating community and providing grants so you can make awesome<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>projects. In the past few years, we've partnered with GitHub to run Summer of Making, hosted&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>the world's longest hackathon on land, and ran Canada's largest high school hackathon.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">|&nbsp;</span>At Hack Club, students aren't just learning, they're shipping.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="text-(--accent)">&nbsp;|</span><br />
                    <span class="text-(--accent)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
                </span>
            </div>
            
            <div class="bg-black/80 absolute bottom-4 right-4 px-1">
                <pre class="font-mono text-xs text-(--text-h) block m-0">
<span class="text-(--accent)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
<span class="text-(--accent)">| </span><span class="font-bold text-(--text)">Hack Club</span>     <span class="font-bold text-(--text)">Resources</span>          <span class="font-bold text-(--text)">Hackanomous</span> <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span>                                             <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/philosophy/" target="_blank" rel="noopener noreferrer">Philosophy</a>    <a class="footer-link" href="https://events.hackclub.com/" target="_blank" rel="noopener noreferrer">Events</a>             Guides      <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/team/" target="_blank" rel="noopener noreferrer">Our Team</a>      <a class="footer-link" href="https://toolbox.hackclub.com/" target="_blank" rel="noopener noreferrer">Toolbox</a>                        <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/jobs/" target="_blank" rel="noopener noreferrer">Jobs (ew)</a>     <a class="footer-link" href="https://hackclub.com/map" target="_blank" rel="noopener noreferrer">Clubs Map</a>                      <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/brand/" target="_blank" rel="noopener noreferrer">Branding</a>      <a class="footer-link" href="https://hackclub.com/conduct/" target="_blank" rel="noopener noreferrer">Code of Conduct</a>                <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/philanthropy/" target="_blank" rel="noopener noreferrer">Donate</a>        <a class="footer-link" href="https://hackclub.com/safeguarding" target="_blank" rel="noopener noreferrer">Safeguarding</a>                   <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">| </span><a class="footer-link" href="https://hackclub.com/imprint/" target="_blank" rel="noopener noreferrer">Imprint</a>       <a class="footer-link" href="https://hackclub.com/privacy/" target="_blank" rel="noopener noreferrer">Privacy &amp; Terms</a>                <span class="text-(--accent)"> |</span>
<span class="text-(--accent)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span></pre>
            </div>
        </section>
    </div>
</div>

<style>
    .floater {
        animation: float 3s ease-in-out infinite;
    }

    .footer-link {
        color: inherit;
        text-decoration: none;
        transition: color 0.2s ease, text-decoration-color 0.2s ease;
    }

    .footer-link:hover {
        color: var(--accent);
        text-decoration: underline;
        text-underline-offset: 2px;
    }

    .footer-link:focus-visible {
        outline: 1px dashed var(--accent);
        outline-offset: 2px;
    }

    @keyframes float {
        0%,
        100% {
            transform: translateY(0);
        }
        50% {
            transform: translateY(12px);
        }
    }
</style>
