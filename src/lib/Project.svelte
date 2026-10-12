<script>
    /** @type {{ label: string; image: string; author: string; href: string }} */
    let { label, image, author, href } = $props();
</script>

<a href={href} target="_blank" rel="noopener noreferrer" aria-label={`View ${label} by ${author}`} class="h-full w-64 shrink-0 border border-(--primary) rounded-2xl overflow-hidden relative transform-gpu motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-300 ease-out">
    <img src={image} class="h-full w-full object-contain" alt={label} />
    <span class="font-mono font-bold text-base text-(--text) tracking-wide absolute bottom-2 left-3">{label}</span>
    <span class="font-mono font-base text-sm text-(--text) absolute top-2 right-3">{author}</span>
</a>
