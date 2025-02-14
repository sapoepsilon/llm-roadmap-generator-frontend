<!-- src/lib/components/ErrorMessage.svelte -->
<script>
    export let message;
</script>

<div class="p-4 mb-4 text-sm text-red-800 bg-red-50 dark:bg-red-900/10 dark:text-red-400 rounded-lg" role="alert">
    <span class="font-medium">Error:</span> {message}
</div>
