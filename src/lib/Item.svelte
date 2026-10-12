<script>
    import { Gift } from "lucide-svelte";

    /** @type {{ label: string; image?: string; bolts: number | string }} */
    let { label, image, bolts } = $props();
</script>

<div class="h-full w-64 shrink-0 border border-(--secondary) rounded-2xl overflow-hidden relative flex flex-col gap-3 p-3 transform-gpu motion-safe:hover:scale-[1.02] motion-safe:hover:rotate-1 transition-all duration-300 ease-out">
    <span class="self-end font-mono font-base text-sm text-(--text) text-right">{typeof bolts === "number" ? `${bolts} bolts` : bolts}</span>
    <div class="flex-1 min-h-0 flex items-center justify-center text-(--secondary)">
        {#if image}
            <img src={image} class="h-full w-full object-contain" alt={label} />
        {:else}
            <Gift size={96} strokeWidth={1.5} aria-hidden="true" />
        {/if}
    </div>
    <span class="shrink-0 font-mono font-bold text-base text-(--text) tracking-wide">{label}</span>
</div>
