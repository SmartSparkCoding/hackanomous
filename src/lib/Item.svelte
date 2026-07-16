<script>
    let { label, image, hours } = $props();
</script>

<div class="h-full w-64 shrink-0 border border-(--accent) rounded-2xl overflow-hidden relative transform-gpu motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-300 ease-out">
    <img src={image} class="h-full w-full object-contain" alt={label} />
    <span class="font-mono font-base text-sm text-(--text) tracking-wide absolute bottom-2 left-3">{label}</span>
    <span class="font-mono font-base text-sm text-(--text) tracking-wide absolute top-2 right-3">~{hours} hours</span>
</div>
