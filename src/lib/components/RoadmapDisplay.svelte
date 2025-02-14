<!-- src/lib/components/RoadmapDisplay.svelte -->
<!-- Cache or save in local db the generated roadmap output -->
<script>
	import Markdown from 'svelte-exmarkdown';
	import { gfmPlugin } from 'svelte-exmarkdown/gfm';

	export let roadmapOutput;
	const plugins = [gfmPlugin()];

	$: ({ idea, initialClarificationQuestions, marketOverview, mvpEpics, taskBreakdown } =
		roadmapOutput || {});
</script>

<div class="mt-6 w-full max-w-4xl space-y-8">
	{#if idea}
		<section class="rounded-lg bg-gradient-to-r from-blue-500 to-purple-600 p-6 shadow-lg">
			<h2 class="mb-4 text-3xl font-bold text-white">Business Idea</h2>
			<p class="text-lg text-white">{idea}</p>
		</section>
	{/if}

	{#if initialClarificationQuestions}
		<section class="rounded-lg bg-white p-6 shadow-lg dark:bg-slate-800">
			<h2 class="mb-4 text-2xl font-bold text-gray-900 dark:text-white">
				Key Questions to Consider
			</h2>
			<div class="prose prose-slate dark:prose-invert max-w-none">
				<Markdown md={initialClarificationQuestions} {plugins} />
			</div>
			<!-- TODO: Add an input to answer these questions -->
		</section>
	{/if}

	{#if marketOverview}
		<section class="rounded-lg bg-emerald-50 p-6 shadow-lg dark:bg-slate-800/50">
			<h2 class="mb-4 text-2xl font-bold text-emerald-900 dark:text-emerald-400">
				Market Analysis
			</h2>
			<div class="prose prose-emerald dark:prose-invert max-w-none">
				<Markdown md={marketOverview} {plugins} />
				<!-- TODO: Add functionality to reference sources used in market analysis -->
			</div>
		</section>
	{/if}

	{#if mvpEpics}
		<section class="rounded-lg bg-amber-50 p-6 shadow-lg dark:bg-slate-800/50">
			<h2 class="mb-4 text-2xl font-bold text-amber-900 dark:text-amber-400">
				MVP Development Epics
			</h2>
			<div class="prose prose-amber dark:prose-invert max-w-none">
				<Markdown md={mvpEpics} {plugins} />
			</div>
		</section>
	{/if}

	{#if taskBreakdown}
		<section class="rounded-lg bg-indigo-50 p-6 shadow-lg dark:bg-slate-800/50">
			<h2 class="mb-4 text-2xl font-bold text-indigo-900 dark:text-indigo-400">
				Detailed Task Breakdown
			</h2>
			<div class="prose prose-indigo dark:prose-invert max-w-none">
				<Markdown md={taskBreakdown} {plugins} />
			</div>
		</section>
	{/if}
</div>
