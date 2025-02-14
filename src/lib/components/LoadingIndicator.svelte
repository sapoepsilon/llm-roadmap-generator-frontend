<!-- src/lib/components/LoadingIndicator.svelte -->
<script>
    export let message = "Loading...";
</script>

<div class="flex items-center justify-center space-x-2">
    <div class="w-4 h-4 rounded-full animate-pulse bg-slate-600"></div>
    <div class="w-4 h-4 rounded-full animate-pulse bg-slate-600 [animation-delay:0.2s]"></div>
    <div class="w-4 h-4 rounded-full animate-pulse bg-slate-600 [animation-delay:0.4s]"></div>
    <span class="text-sm text-slate-600 dark:text-slate-400">{message}</span>
</div>
