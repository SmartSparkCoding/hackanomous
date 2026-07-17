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

    import itemRaspberryPi from "./assets/items/raspberry-pi.png";

    import Project from "./lib/Project.svelte";
    import Item from "./lib/Item.svelte";
    import Question from "./lib/Question.svelte";

    /** @type {HTMLElement | undefined} */
    let overlay;
    /** @type {HTMLDivElement | undefined} */
    let horizontalScroller;
    /** @type {HTMLElement | undefined} */
    let horizontalSection;
    /** @type {HTMLDivElement | undefined} */
    let projectScroller;
    /** @type {HTMLDivElement | undefined} */
    let prizeScroller;
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
    const scrollerItemIndexes = Array.from({ length: 32 }, (_, index) => index);

    let activeEvent = $derived(events[carouselIndex]);

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
        // smoothie = ScrollSmoother.create({
        //     smooth: 1,
        //     effects: true,
        //     normalizeScroll: true,
        // });

        const scrollerEl = horizontalScroller;
        const scrollerSectionEl = horizontalSection;
        const projectScrollerEl = projectScroller;
        const prizeScrollerEl = prizeScroller;
        const horizontalScrollHold = { progress: 0 };
        const getHorizontalScrollDistance = () => Math.max(0, (scrollerEl?.scrollWidth ?? 0) - (scrollerSectionEl?.clientWidth ?? 0));
        const horizontalScrollTimeline =
            scrollerEl &&
            scrollerSectionEl &&
            projectScrollerEl &&
            prizeScrollerEl &&
            gsap
                .timeline({
                    scrollTrigger: {
                        trigger: scrollerSectionEl,
                        start: "top top",
                        end: () => `+=${getHorizontalScrollDistance() * 1.08}`,
                        scrub: true,
                        pin: true,
                        anticipatePin: 1,
                        invalidateOnRefresh: true,
                    },
                })
                .to(horizontalScrollHold, {
                    progress: 1,
                    duration: 0.05,
                    ease: "none",
                })
                .addLabel("travel")
                .to(
                    scrollerEl,
                    {
                        x: () => -getHorizontalScrollDistance(),
                        duration: 1,
                        ease: "none",
                    },
                    "travel",
                )
                .to(
                    projectScrollerEl,
                    {
                        x: () => window.innerWidth * 1.0, // +counterstage motion: as the user scrolls the content to the left, we scroll items 50% to the right to net appear a 50% scroll left. cool, right?
                        duration: 1,
                        ease: "none",
                    },
                    "travel",
                )
                .to(
                    prizeScrollerEl,
                    {
                        x: () => window.innerWidth * -0.5, // -withstage motion: pretty much same as above note, just inverted vfx rate
                        duration: 1,
                        ease: "none",
                    },
                    "travel",
                )
                .to(
                    horizontalScrollHold,
                    {
                        progress: 2,
                        duration: 0.05,
                        ease: "none",
                    },
                    "travel+=1",
                );

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
            horizontalScrollTimeline?.scrollTrigger?.kill();
            horizontalScrollTimeline?.kill();

            stepStateTrigger?.kill();

            stepProgressPin?.kill();

            stepsVisualPinTrigger?.kill();

            // disable smoothscroll
            // smoothie.kill();
        };
    });
</script>

<div id="smooth-wrapper" class="relative overflow-x-clip bg-[linear-gradient(150deg,#080E12,#0B1618)]">
    <!-- cool visuals overlay -->
    <div bind:this={overlay} class="fixed inset-0 w-dvw h-dvh pointer-events-none z-0"></div>

    <div id="smooth-content" class="relative z-10">
        <!-- hackclub icon -->
        <!-- <section class="absolute top-6 left-1/2 -translate-x-1/2 z-50 flex items-center gap-2">
            <h6 class="text-(--primary-bg) ml-[26.13px] tracking-wide">a hack</h6>
            <a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="block">
                <img src={hackclub} class="cursor-pointer transition-all duration-200 ease-out hover:-translate-y-1 hover:scale-105 hover:drop-shadow-xl active:scale-95 active:-translate-y-1 active:duration-150" width="40" alt="Hack Club logo" />
            </a>
            <h6 class="text-(--primary-bg) tracking-wide">club ysws</h6>
        </section> -->

        <!-- orpheus flag -->
        <section class="absolute top-0 left-6">
            <a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="block">
                <img src={orpheus} class="cursor-pointer" width="180" alt="Orpheus flag" />
            </a>
        </section>

        <!-- hackanomous presents -->
        <!-- <section class="absolute top-[50dvh] left-1/2 translate-x-[-23rem] translate-y-[-11rem] -rotate-[15deg] z-1 underline underline-offset-2 decoration-(--primary-bg) z-20">
            <h6 class="font-content text-(--primary-bg) tracking-wider">hackanomous presents</h6>
        </section> -->

        <!-- landing -->
        <section class="relative min-h-dvh flex justify-center items-center py-12 px-4">
            <div class="relative">
                <div class="w-fit border border-(--primary-border) rounded-2xl px-16 py-2 relative z-10 -ml-24 -rotate-5 bg-[linear-gradient(175deg,var(--bg)_0%,#0B161850_33%,#080E1280_100%)]">
                    <h3 class="font-content font-extralight text-sm text-(--primary-text-h) tracking-widest">coming soon!</h3>
                </div>
                <div class="flex">
                    <div class="text-right">
                        <h1 class="font-heading font-regular text-7xl text-(--primary) mt-4">
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
                        <input id="rsvp-input" class="peer w-full px-4 font-mono font-light border-2 border-solid border-(--primary) text-(--primary-text-h) rounded-xl py-3 focus:outline-none focus:border-(--primary-hover) focus:shadow-[0_0_67px_color-mix(in_srgb,var(--primary-hover)_6.7%,transparent)] bg-transparent placeholder:text-(--primary-bg) placeholder-opacity-100" style="transition: box-shadow 0.35s ease-out, border-color 0.15s ease-out, background-color 0.15s ease-out;" type="text" placeholder="you@example.com" />
                        <label for="rsvp-input" class="absolute left-3 -top-2.5 px-1 font-mono text-xs font-semibold text-(--primary-hover) bg-[#0a1215] opacity-0 translate-y-2 pointer-events-none transition-all duration-100 peer-focus:opacity-100 peer-focus:translate-y-0 rounded-sm border-2">email</label>
                    </div>
                    <button onclick={() => window.open("https://hackanomous-rsvp.fillout.com/t/4oPTMjqFuaus", "_blank")} class="font-mono font-semibold border-2 border-solid border-(--primary) bg-(--primary) text-(--bg) rounded-xl px-10 py-3 cursor-pointer hover:bg-transparent hover:text-(--primary) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--primary-hover)_6.7%,transparent)]" style="transition: box-shadow 0.35s ease-out, border-color 0.15s ease-out, background-color 0.15s ease-out, color 0.15s ease-out;">RSVP!</button>
                </div>
                <h6 class="font-mono font-light text-xs text-center text-(--primary-text-h) tracking-widest mt-4">
                    ages 13-18 only. <span class="underline underline-offset-2">Sept 1</span> to <span class="underline underline-offset-2">Jan 1</span>.
                </h6>
                <div class="flex flex-col absolute bottom-14 -right-3 -rotate-5 font-mono text-xs text-(--primary-bg)">
                    <span class="text-right">with lots of love,</span>
                    <span class="text-right">jacob abby techno jenny &lt;3</span>
                </div>
            </div>
            
            <!-- scroll down -->
            <section class="absolute bottom-8 left-1/2 -translate-x-1/2 text-(--primary-bg)">
                <div class="flex flex-row items-center gap-3 floater">
                    <Mouse size={24} />
                    <span class="font-content font-light text-base tracking-widest">SCROLL DOWN</span>
                </div>
            </section>
        </section>

        <!-- ysws overview -->
        <div class="top-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--primary) to-transparent opacity-67 z-10 relative"></div>
            <section class="relative min-h-dvh px-6 md:px-12 xl:px-24 py-24 pb-36 flex flex-col justify-center items-center bg-black/80">

            <div class="max-w-6xl mx-auto w-full">
                <h1 class="font-heading text-3xl mb-8 tracking-wide">what is <span class="font-mono font-bold text-4xl text-(--primary)">Hackanomous</span>?</h1>
                <p class="font-content leading-relaxed"><span class="font-mono font-bold text-(--primary)">Hackanomous</span> is a <span class="font-mono font-bold text-(--primary)">YSWS</span> program where you design and ship <span class="font-mono font-bold text-(--primary)">AI-driven</span> personal projects: hardware or software.<br>We walk you through building your own projects while exploring both effective and ineffective usecases of AI, and ship you free rewards!<br><br>We're even hosting a hackathon in <span class="font-mono font-bold text-(--primary)">Islambad, Pakistan</span> to conclude the event! Qualify by earning enough hours to score an invite!</p>

                <div class="mt-14 w-full">
                    <div class="group relative block w-full origin-center transform-gpu overflow-hidden rounded-2xl border-2 border-(--primary-border) bg-black/70 aspect-16/10 sm:aspect-video lg:aspect-21/9 focus-within:border-(--primary) focus-within:shadow-[0_0_67px_color-mix(in_srgb,var(--primary-hover)_5%,transparent)] hover:border-(--primary) hover:shadow-[0_0_67px_color-mix(in_srgb,var(--primary-hover)_4%,transparent)] motion-safe:hover:scale-[1.018] motion-safe:hover:rotate-[0.8deg] transition-all duration-500 ease-out will-change-transform">
                        <a href={activeEvent.link} target="_blank" rel="noopener noreferrer" aria-label={`Open ${activeEvent.label}`} class="absolute inset-0 z-10 cursor-pointer focus:outline-none"></a>
                        {#key carouselIndex}
                            <img transition:fade={{ duration: 200 }} src={activeEvent.image} alt={activeEvent.label} class="absolute inset-0 h-full w-full object-cover transition-transform duration-700 ease-out group-hover:scale-[1.03]" />
                        {/key}
                        <div class="absolute inset-0 bg-[linear-gradient(180deg,rgba(8,14,18,0)_28%,rgba(8,14,18,0.88)_100%)]"></div>
                        <div class="absolute inset-0 border border-white/10 rounded-[14px] pointer-events-none"></div>

                        <div class="pointer-events-none absolute left-4 right-4 bottom-4 z-20 md:left-6 md:right-6 md:bottom-6 flex flex-col gap-4 md:flex-row md:items-end md:justify-between">
                            <div>
                                <span class="font-content font-light text-sm tracking-widest">previous events!</span>
                                <h2 class="font-mono font-medium text-xl md:text-3xl text-(--text) leading-tight">{activeEvent.label}</h2>
                            </div>
                            <div class="pointer-events-auto flex gap-2 translate-y-2" role="group" aria-label="Event carousel controls">
                                {#each events as event, index (event.label)}
                                    <button type="button" onclick={() => (carouselIndex = index)} aria-label={`Show ${event.label}`} aria-pressed={index === carouselIndex} class="flex h-5 w-10 items-center rounded-full border-none bg-transparent p-0 opacity-80 transition-opacity duration-200 hover:opacity-100 focus:outline-none focus-visible:shadow-[0_0_0_2px_var(--primary)] cursor-pointer">
                                        <span class={`h-1.5 w-full rounded-full transition-all duration-200 ${index === carouselIndex ? "bg-(--primary)" : "bg-(--primary-bg)/50"}`}></span>
                                    </button>
                                {/each}
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ...so how does this work? -->
            <div class="absolute bottom-8 left-1/2 -translate-x-1/2 floater flex flex-col items-center">
                <span class="font-content font-light text-base tracking-widest">...so how does this work?</span>
                <ChevronDown size={24}/>
            </div>

            <div class="absolute bottom-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--primary) to-transparent opacity-67 z-10"></div>
        </section>

        <section bind:this={stepsSection} class="min-h-dvh flex flex-col items-center overflow-hidden w-full" id="steps-section">
            <div class="w-full max-w-7xl mx-auto flex flex-row relative py-32 px-6 lg:[--steps-copy-width:calc((min(100dvw,80rem)-12rem)/2)] lg:w-[calc(var(--steps-copy-width)+var(--steps-copy-width)-8rem)] lg:max-w-none lg:px-0">
                
                <!-- left content scrolling -->
                <div class="w-1/2 flex flex-col relative pr-20 lg:w-(--steps-copy-width) lg:shrink-0">
                    
                    <!-- steps progress bar -->
                    <div class="absolute inset-y-0 left-0 w-1 z-20">
                        <div id="steps-progress-wrapper" class="w-full h-[50vh] min-h-[50dvh] bg-(--primary-border)">
                            <div id="step-progress" class="w-full h-1/4 bg-(--primary) shadow-[0_0_20px_color-mix(in_srgb,var(--primary)_3%,transparent)]" style="transform: translateY(0%);"></div>
                        </div>
                    </div>

                    <!-- steps -->
                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 0} class:opacity-30={activeStepIndex !== 0}>
                        <div class="absolute">
                            <!-- <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">1</h1> -->
                        </div>
                        <h2 class="font-content font-medium text-4xl text-(--text) mb-4">Inspire to create</h2>
                        <p class="font-content font-light text-lg text-(--text) leading-relaxed tracking-wide">
                            Brainstorm an innovative and interesting idea! Explore practical scenarios in which AI could actually make a productive difference.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 1} class:opacity-30={activeStepIndex !== 1}>
                        <div class="absolute">
                            <!-- <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">2</h1> -->
                        </div>
                        <h2 class="font-content font-medium text-4xl text-(--text) mb-4">Build your project</h2>
                        <p class="font-content font-light text-lg text-(--text) leading-relaxed tracking-wide">
                            Bring your idea to life through code and hardware! If you're making a hardware project, you're eligible for a funding grant.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 2} class:opacity-30={activeStepIndex !== 2}>
                        <div class="absolute">
                            <!-- <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">3</h1> -->
                        </div>
                        <h2 class="font-content font-medium text-4xl text-(--text) mb-4">Ship to the universe</h2>
                        <p class="font-content font-light text-lg text-(--text) leading-relaxed tracking-wide">
                            Show off your finished project! Also, check out what other amazing projects by other teenage inventors have been building and shipping.
                        </p>
                    </div>

                    <div class="step-item min-h-[60dvh] flex flex-col justify-center pl-12 transition-opacity duration-700 ease-out" class:opacity-100={activeStepIndex === 3} class:opacity-30={activeStepIndex !== 3}>
                        <div class="absolute">
                            <!-- <h1 class="relative font-mono font-medium text-7xl -left-37.75 top-9.25">4</h1> -->
                        </div>
                        <h2 class="font-content font-medium text-4xl text-(--text) mb-4">Claim your rewards</h2>
                        <p class="font-content font-light text-lg text-(--text) leading-relaxed tracking-wide">
                            You did it! In exchange for helping take AI to the next level, receive Bolts to get DDR5 RAM, AI credits, or perhaps an RTX 5090!
                        </p>
                    </div>

                </div>

                <!-- right side -->
                <div class="w-1/2 relative h-full lg:-ml-32 lg:w-(--steps-copy-width) lg:shrink-0">
                    <div bind:this={stepsVisual} class="relative flex aspect-square w-full items-center justify-center overflow-hidden lg:aspect-auto lg:h-[calc(var(--steps-copy-width)+4rem)]">
                        {#key activeStepIndex}
                            <section class="absolute inset-0 flex flex-col items-center gap-3 px-8 text-center font-mono text-(--primary-text-h) will-change-opacity">
                                {#if activeStepIndex === 0}
                                    <div class="relative h-[78%] w-[78%] lg:w-[calc(78%+3.12rem)]">
                                        <img src={mascotDark} alt="anomaly having an idea" class="absolute bottom-[15%] left-[calc(50%+18px)] h-[58%] -translate-x-1/2 object-contain" />
                                        <Lightbulb class="absolute left-1/2 top-[15%] h-24 w-24 -translate-x-1/2 text-(--primary) lg:h-28 lg:w-28" strokeWidth={1.8} />
                                    </div>
                                {:else if activeStepIndex === 1}
                                    <div class="relative h-[78%] w-[78%] lg:w-[calc(78%+3.12rem)]">
                                        <Hammer class="absolute bottom-[48%] left-[calc(50%+136px)] h-24 w-24 -translate-x-1/2 text-(--primary) lg:h-28 lg:w-28" strokeWidth={1.9} />
                                        <img src={mascotDark} alt="anomaly holding a hammer" class="absolute bottom-[23%] left-[calc(50%-24px)] h-[56%] -translate-x-1/2 object-contain" />
                                    </div>
                                {:else if activeStepIndex === 2}
                                    <div class="relative h-[68%] w-[84%] lg:w-[calc(84%+3.36rem)]">
                                        <Truck class="absolute bottom-[22%] left-1/2 h-[52%] w-[86%] -translate-x-1/2 text-(--primary)" strokeWidth={1.8} />
                                        <img src={mascotDark} alt="anomaly logo on truck side" class="absolute bottom-[42%] left-[42.5%] h-[20%] -translate-x-1/2 object-contain" />
                                    </div>
                                {:else}
                                    <div class="relative h-[78%] w-[48%] lg:w-[calc(48%+1.92rem)]">
                                        <img src={mascotDark} alt="anomaly with a reward" class="absolute bottom-[16%] left-[calc(50%-18px)] h-[58%] -translate-x-1/2 object-contain" />
                                        <Trophy class="absolute left-[calc(50%-42px)] top-[18%] h-24 w-24 -translate-x-1/2 text-(--primary) lg:h-28 lg:w-28" strokeWidth={1.8} />
                                    </div>
                                {/if}
                            </section>
                        {/key}
                    </div>
                </div>

            </div>
        </section>

        <!-- horizontal section -->
        <section bind:this={horizontalSection} class="min-h-dvh flex flex-col relative overflow-hidden bg-black/80">
            <!-- top border strip glow -->
            <div class="absolute top-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--primary) to-transparent opacity-67"></div>

            <div class="w-full overflow-hidden flex-1">
                <div bind:this={horizontalScroller} class="horizontal-scroller h-full absolute overflow-hidden">
                    <!-- left content -->
                    <!-- TODO: rewrite ts slop -->
                    <div class="absolute top-1/2 left-[5dvw] -translate-y-[calc(50%+28px)] w-[85dvw] sm:w-[70dvw] md:w-[60dvw] lg:w-[45dvw] z-10 flex flex-col items-start text-left">
                        <h2 class="font-mono font-medium text-4xl md:text-6xl text-(--text) leading-tight">
                            <span class="italic">DE</span>SLOP THE<br />
                            <span class="font-mono font-bold inline-block bg-clip-text text-transparent bg-[linear-gradient(90deg,var(--primary)_0%,color-mix(in_srgb,var(--primary)_67%,transparent)_100%)]">WORLD.</span>
                        </h2>

                        <p class="font-mono font-normal text-lg md:text-xl leading-relaxed text-(--primary-text-h) mt-6 max-w-2xl">
                            the AI bubble might just be about to pop.<br />
                            <span class="font-bold text-(--primary-text-l)">YOUR MISSION:</span> build projects incorporating AI that solve <u><span class="font-bold text-(--primary-text-l)">real-world</span></u> problems.
                        </p>

                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-6 mt-10 w-full max-w-3xl">
                            <div class="border border-(--primary-border) p-6 rounded-2xl origin-center transform-gpu motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-300 ease-out will-change-transform group cursor-default">
                                <h4 class="font-heading text-3xl text-(--primary) mb-2 text-left">software</h4>
                                <p class="font-content text-sm font-light text-(--primary-text-h) group-hover:text-(--text) transition-colors duration-300">Build software that implements AI or ML to earn Bolts! Use them to buy Raspberry Pis, API credits, RAM & GPU grants, and more!</p>
                            </div>
                            <div class="border border-(--primary-border) p-6 rounded-2xl origin-center transform-gpu motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-300 ease-out will-change-transform group cursor-default">
                                <h4 class="font-heading text-3xl text-(--primary) mb-2 text-left">hardware</h4>
                                <p class="font-content text-sm font-light text-(--primary-text-h) group-hover:text-(--text) transition-colors duration-300">Design hardware that implements AI or ML and receive funding to build it! Earn bolts for your physical work!</p>
                            </div>
                        </div>

                        <div class="flex gap-5 mt-8">
                            <button onclick={() => gsap.to(window, { duration: .67, scrollTo: { y: '#faq', autoKill: true }, ease: 'power2.inOut' })} class="font-mono font-semibold border-2 border-solid border-(--primary) text-(--primary) hover:bg-(--primary) hover:text-(--bg) rounded-xl px-12 py-4 cursor-pointer focus:outline-none hover:shadow-[0_0_30px_color-mix(in_srgb,var(--primary)_30%,transparent)] transition-all duration-150 tracking-wide text-lg"> LEARN MORE </button>
                            <button onclick={() => gsap.to(window, { duration: 1, scrollTo: { y: 0, autoKill: true }, ease: 'power2.inOut' })} class="font-mono font-semibold border-2 border-solid border-(--primary) text-(--primary) hover:bg-(--primary) hover:text-(--bg) rounded-xl px-12 py-4 cursor-pointer focus:outline-none hover:shadow-[0_0_30px_color-mix(in_srgb,var(--primary)_30%,transparent)] transition-all duration-150 tracking-wide text-lg"> REGISTER NOW </button>
                        </div>
                    </div>

                    <!-- projects -->
                    <div class="project-lane z-10 relative rotate-25 overflow-hidden">
                        <div bind:this={projectScroller} class="h-full min-w-full w-max border border-(--primary-border) flex p-4 gap-4 rounded-bl-3xl">
                            <!-- TODO: fill projects -->
                            <!-- TODO: differentiate from items/shop bc i lit just copypasted it for now -->
                            {#each scrollerItemIndexes as index (`project-${index}`)}
                                <Project label="Raspberry Pi 6 or 7" image={itemRaspberryPi} hours={67} />
                            {/each}
                        </div>
                    </div>

                    <!-- prizes -->
                    <div class="prize-lane z-10 relative rotate-25 overflow-hidden">
                        <div bind:this={prizeScroller} class="h-full min-w-full w-max border border-(--secondary-bg) flex p-4 gap-4">
                            <!-- TODO: fill shop -->
                            {#each scrollerItemIndexes as index (`prize-${index}`)}
                                <Item label="Raspberry Pi 6 or 7" image={itemRaspberryPi} hours={67} />
                            {/each}
                        </div>
                    </div>

                    <!-- right content -->
                    <!-- TODO: rewrite ts slop -->
                    <div class="absolute top-[42%] right-[5dvw] -translate-y-[calc(50%+28px)] w-[85dvw] sm:w-[70dvw] md:w-[60dvw] lg:w-[45dvw] z-10 flex flex-col items-end text-right selection:!bg-(--primary)">
                        <h2 class="font-mono font-medium text-4xl md:text-6xl text-(--text) leading-tight">
                            <span class="italic">DE</span>SLOP THE<br />
                            <span class="font-mono font-bold inline-block bg-clip-text text-transparent bg-[linear-gradient(90deg,var(--secondary)_0%,color-mix(in_srgb,var(--secondary)_67%,transparent)_100%)]">WORLD.</span>
                        </h2>

                        <p class="font-mono font-normal text-lg md:text-xl leading-relaxed text-(--secondary-text) mt-6 max-w-2xl">
                            the AI bubble might just be about to pop.<br />
                            <span class="font-bold text-(--secondary-text)">YOUR MISSION:</span> build projects incorporating AI that solve <u><span class="font-bold text-(--secondary-text)">real-world</span></u> problems.
                        </p>

                        <div class="flex gap-5 mt-8">
                            <button onclick={() => gsap.to(window, { duration: 1, scrollTo: { y: 0, autoKill: true }, ease: 'power2.inOut' })} class="font-mono font-semibold border-2 border-solid border-(--secondary) text-(--secondary) hover:bg-(--secondary) hover:text-(--bg) rounded-xl px-12 py-4 cursor-pointer focus:outline-none hover:shadow-[0_0_30px_color-mix(in_srgb,var(--secondary)_30%,transparent)] transition-all duration-150 tracking-wide text-lg"> SEE THE FULL SHOP </button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- bottom border strip glow -->
            <div class="absolute bottom-0 left-0 w-full h-0.5 bg-linear-to-r from-transparent via-(--primary) to-transparent opacity-67"></div>
        </section>

        <!-- standard FAQ and closing info -->
        <section id="faq" class="min-h-dvh px-6 md:px-12 xl:px-24 py-24 pb-55 flex flex-col justify-start items-center relative">
            <!-- pb-[220px] = calc(16px + 180px + 24px) -->
            <!-- faq -->
            <div class="max-w-7xl mx-auto w-full">
                <div class="relative w-full mb-12">
                    <h1 class="font-heading font-regular text-5xl md:text-6xl text-(--text) w-full relative z-10">FAQ</h1>
                    <!-- real text -->
                    <div class="absolute top-0 left-0 font-heading font-regular text-5xl md:text-6xl text-(--primary)" style="transform: translate(6px, 3px);">FAQ</div>
                    <!-- offset text -->
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-x-12 lg:gap-x-24 gap-y-0 w-full">
                    <!-- column 1 -->
                    <div class="flex flex-col">
                        <Question question="Who can participate?">
                            <!-- vetted answer -->
                            <p>Anyone between the ages of <span class="font-mono font-bold text-(--primary)">13</span> and <span class="font-mono font-bold text-(--primary)">18</span> (inclusive) can participate in Hackanomous!</p>
                        </Question>
                        <Question question="What projects can I build?">
                            <!-- vetted answer -->
                            <p>You can build any project incorporating <span class="font-mono font-bold text-(--primary)">AI</span> or <span class="font-mono font-bold text-(--primary)">ML</span>! We want to see your innovation shine, whether it's software, hardware, or something COMPLETELY new and unique.</p>
                        </Question>
                        <Question question="I don't know anything about AI/ML!">
                            <!-- vetted answer -->
                            <p><span class="font-mono font-bold text-(--primary)">No problem!</span> We have resources and mentors to help you learn. Plus, the best way to learn is by building, so dive in and start experimenting!</p>
                        </Question>
                        <Question question="What are the prizes?">
                            <!-- vetted answer -->
                            <p>We have a variety of exciting prizes for our participants! Shipped projects will receive <span class="font-mono font-bold text-(--primary)">Bolts</span>, which can be redeemed for Raspberry Pis, AI credits, RAM & GPU grants, and more. We might even send you an <span class="font-mono font-bold text-(--primary)">RTX 5090</span>!</p>
                        </Question>
                        <Question question="Is this legit? What's Hack Club?">
                            <!-- vetted answer -->
                            <p><a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="font-mono font-bold underline text-(--primary)">Hack Club</a> is the world's largest community of teenage makers, and a 501(c)(3) nonprofit. Hack Club is supported by donations from tech companies like GitHub and individuals like Michael Dell. Hack Club is fiscally transparent.</p>
                        </Question>
                    </div>

                    <!-- column 2 -->
                    <div class="flex flex-col">
                        <Question question="How do I register?">
                            <!-- vetted answer -->
                            <p>Registration for Hackanomous is simple! Just scroll up and click the <span class="font-mono font-bold text-(--primary)">RSVP</span> button to fill out the registration form.</p>
                        </Question>
                        <Question question="How many projects can I build?">
                            <!-- vetted answer -->
                            <p>You can build as many projects as you like, but we encourage you to focus on <span class="font-mono font-bold text-(--primary)">quality over quantity</span>. Your rewards <span class="font-mono font-bold text-(--primary)">scale exponentially</span> with time, so really focus on building something amazing!</p>
                        </Question>
                        <Question question="How do I fund hardware?">
                            <!-- vetted answer -->
                            <p>Don't worry! You can get up to <span class="font-mono font-bold text-(--primary)">$100</span> <span class="font-mono font-bold text-(--primary)">(TODO: TBD)</span> to buy parts to build your hardware project.</p>
                        </Question>
                        <Question question="Is this free?">
                            <!-- vetted answer -->
                            <p><span class="font-mono font-bold text-(--primary)">Yes!</span> Hackanomous is <span class="font-mono font-bold text-(--primary)">100%</span> free - all the prizes are donated by sponsors or paid for by us! <span class="font-mono text-sm text-(--primary)">*customs may occur outside the US!</span></p>
                        </Question>
                        <Question question="I have more questions...">
                            <!-- vetted answer -->
                            <p>Ask us in <span class="font-mono font-bold text-(--primary)">#hackanomous-help</span> in Slack!</p>
                            <!-- switch out when we get an hc email -->
                            <!-- <p>Ask us in <span class="font-mono font-bold text-(--primary)">#hackanomous-help</span> in Slack, or email us at <a href="mailto:hackanomous@hackclub.com" class="font-mono font-bold underline text-(--primary)">hackanomous@hackclub.com</a>!</p> -->
                        </Question>
                    </div>
                </div>
            </div>

            <!-- closing info -->
            <div class="bg-black/80 absolute bottom-4 left-4 px-1">
<pre class="font-mono text-xs text-(--primary-text-h) block">
<span class="text-(--primary)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
<span class="text-(--primary)">| </span><span class="font-bold text-(--text)"># Built by <a class="underline underline-offset-1" href="https://hackclub.enterprise.slack.com/team/U0A76B70A3V" target="_blank" rel="noopener noreferrer">technodot</a> @ Hack Club</span>                                                               <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>                                                                                               <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>Hack Club is a 501(c)(3) nonprofit and network of 67k+ technical high schoolers. We believe you<span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>learn best by building so we're creating community and providing grants so you can make awesome<span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>projects. In the past few years, we've partnered with GitHub to run Summer of Making, hosted   <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>the world's longest hackathon on land, and ran Canada's largest high school hackathon.         <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>                                                                                               <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>At Hack Club, students aren't just learning, they're shipping.                                 <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
</pre>
            </div>
            
            <div class="bg-black/80 absolute bottom-4 right-4 px-1">
<pre class="font-mono text-xs text-(--primary-text-h) block m-0">
<span class="text-(--primary)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
<span class="text-(--primary)">| </span><span class="font-bold text-(--text)">Hack Club</span>     <span class="font-bold text-(--text)">Resources</span>          <span class="font-bold text-(--text)">Hackanomous</span> <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span>                                             <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/philosophy/" target="_blank" rel="noopener noreferrer">Philosophy</a>    <a class="footer-link" href="https://events.hackclub.com/" target="_blank" rel="noopener noreferrer">Events</a>             Guides      <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/team/" target="_blank" rel="noopener noreferrer">Our Team</a>      <a class="footer-link" href="https://toolbox.hackclub.com/" target="_blank" rel="noopener noreferrer">Toolbox</a>                        <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/jobs/" target="_blank" rel="noopener noreferrer">Jobs (ew)</a>     <a class="footer-link" href="https://hackclub.com/map" target="_blank" rel="noopener noreferrer">Clubs Map</a>                      <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/brand/" target="_blank" rel="noopener noreferrer">Branding</a>      <a class="footer-link" href="https://hackclub.com/conduct/" target="_blank" rel="noopener noreferrer">Code of Conduct</a>                <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/philanthropy/" target="_blank" rel="noopener noreferrer">Donate</a>        <a class="footer-link" href="https://hackclub.com/safeguarding" target="_blank" rel="noopener noreferrer">Safeguarding</a>                   <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">| </span><a class="footer-link" href="https://hackclub.com/imprint/" target="_blank" rel="noopener noreferrer">Imprint</a>       <a class="footer-link" href="https://hackclub.com/privacy/" target="_blank" rel="noopener noreferrer">Privacy &amp; Terms</a>                <span class="text-(--primary)"> |</span>
<span class="text-(--primary)">+-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-+</span>
</pre>
            </div>
        </section>
    </div>
</div>

<style>
    .horizontal-scroller {
        --viewport-height-growth: clamp(0px, calc(100dvh - 960px), 1200px);
        --lane-height: 384px;
        --lane-width: calc(200rem + var(--viewport-height-growth) * 2.3662015832);
        --lane-origin-x: 100rem;
        --project-lane-y: 13.25rem;
        --prize-lane-y: 236px;
        --prize-lane-x: 900px;
        width: calc(var(--prize-lane-x) + var(--lane-width));
    }

    .prize-lane, .project-lane {
        width: var(--lane-width);
        height: var(--lane-height);
        transform-origin: var(--lane-origin-x) 50%;
    }

    .project-lane {
        top: var(--project-lane-y);
    }

    .prize-lane {
        /* (sin 25deg, −cos 25deg) */
        top: calc(var(--prize-lane-y) - var(--lane-height) - 0.906307787rem);
        left: calc(var(--prize-lane-x) + 0.422618262rem);
    }

    .floater {
        animation: float 3s ease-in-out infinite;
    }

    .footer-link {
        color: inherit;
        text-decoration: none;
        transition: color 0.2s ease, text-decoration-color 0.2s ease;
    }

    .footer-link:hover {
        color: var(--primary);
        text-decoration: underline;
        text-underline-offset: 2px;
    }

    .footer-link:focus-visible {
        outline: 1px dashed var(--primary);
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
