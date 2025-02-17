<!-- src/lib/components/RoadmapDisplay.svelte -->
<script>
	import Markdown from 'svelte-exmarkdown';
	import { gfmPlugin } from 'svelte-exmarkdown/gfm';

	export let roadmapOutput;
	const plugins = [gfmPlugin()];

	$: ({
		idea,
		initialClarificationQuestions,
		marketOverview,
		marketOverviewGroundingMetadata,
		mvpEpics,
		taskBreakdown,
		conversationHistory = []
	} = roadmapOutput || {});

	$: console.log('Debug - roadmapOutput:', roadmapOutput);

	// Format conversation role for display
	console.log(`conversationHistory:`, conversationHistory);
	const formatRole = (role) => {
		return role.charAt(0).toUpperCase() + role.slice(1);
	};
</script>

<div class="mt-6 w-full max-w-4xl space-y-8">
	{#if idea !== undefined}
		<section
			class="rounded-lg bg-gradient-to-r from-blue-500 to-purple-600 p-6 shadow-lg transition-all hover:shadow-xl"
		>
			<h2 class="mb-4 text-3xl font-bold text-white">Business Idea</h2>
			<p class="text-lg text-white">{idea}</p>
		</section>
	{/if}

	{#if initialClarificationQuestions !== undefined}
		<section
			class="rounded-lg bg-white p-6 shadow-lg transition-all hover:shadow-xl dark:bg-slate-800"
		>
			<h2 class="mb-4 text-2xl font-bold text-gray-900 dark:text-white">
				Key Questions to Consider
			</h2>
			<div class="prose prose-slate dark:prose-invert max-w-none">
				<Markdown md={initialClarificationQuestions} {plugins} />
			</div>
		</section>
	{/if}

	{#if marketOverview !== undefined}
		<section
			class="rounded-lg bg-emerald-50 p-6 shadow-lg transition-all hover:shadow-xl dark:bg-slate-800/50"
		>
			<h2 class="mb-4 text-2xl font-bold text-emerald-900 dark:text-emerald-400">
				Market Analysis
			</h2>
			<div class="prose prose-emerald dark:prose-invert max-w-none">
				<Markdown md={marketOverview} {plugins} />
				{#if marketOverviewGroundingMetadata}
					<div class="mt-4 text-sm text-emerald-600 dark:text-emerald-400">
						<p class="font-semibold">Sources:</p>
						<ul class="list-disc pl-5">
							{#each Object.entries(marketOverviewGroundingMetadata) as [source, metadata]}
								<li>{source}: {metadata}</li>
							{/each}
						</ul>
					</div>
				{/if}
			</div>
		</section>
	{/if}

	{#if mvpEpics !== undefined}
		<section
			class="rounded-lg bg-amber-50 p-6 shadow-lg transition-all hover:shadow-xl dark:bg-slate-800/50"
		>
			<h2 class="mb-4 text-2xl font-bold text-amber-900 dark:text-amber-400">
				MVP Development Epics
			</h2>
			<div class="prose prose-amber dark:prose-invert max-w-none">
				<Markdown md={mvpEpics} {plugins} />
			</div>
		</section>
	{/if}

	{#if taskBreakdown !== undefined}
		<section
			class="rounded-lg bg-indigo-50 p-6 shadow-lg transition-all hover:shadow-xl dark:bg-slate-800/50"
		>
			<h2 class="mb-4 text-2xl font-bold text-indigo-900 dark:text-indigo-400">
				Detailed Task Breakdown
			</h2>
			<div class="prose prose-indigo dark:prose-invert max-w-none">
				<Markdown md={taskBreakdown} {plugins} />
			</div>
		</section>
	{/if}

	{#if conversationHistory && conversationHistory.length > 0}
		<section
			class="rounded-lg bg-gray-50 p-6 shadow-lg transition-all hover:shadow-xl dark:bg-slate-800/50"
		>
			<h2 class="mb-4 text-2xl font-bold text-gray-900 dark:text-gray-400">Conversation History</h2>
			<div class="space-y-4">
				{#each conversationHistory as { role, parts }, i}
					{#if role && parts && role !== 'user'}
						<div
							class="rounded-lg p-4 {role === 'assistant'
								? 'bg-blue-50 dark:bg-blue-900/20'
								: 'bg-gray-100 dark:bg-gray-700/20'}"
						>
							<p
								class="mb-2 text-sm font-semibold {role === 'assistant'
									? 'text-blue-600 dark:text-blue-400'
									: 'text-gray-600 dark:text-gray-400'}"
							>
								{formatRole(role)}
							</p>
							<div class="prose prose-sm dark:prose-invert max-w-none">
								<Markdown md={parts} {plugins} />
							</div>
						</div>
					{/if}
				{/each}
			</div>
		</section>
	{/if}
</div>
