<!-- src/routes/+page.svelte -->
<script>
	import axios from 'axios';
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Button } from '$lib/components/ui/button';

	let ideaDescription = '';
	let roadmapOutput = null;
	let conversationId = null;
	let loading = false;
	let errorMessage = '';

	async function generateRoadmap() {
		loading = true;
		errorMessage = '';
		roadmapOutput = null;
		conversationId = null;

		try {
			const response = await axios.post('http://localhost:3000/generate-roadmap', {
				ideaDescription
			});
			conversationId = response.data.conversationId;
			console.log('Received conversationId:', conversationId);
			roadmapOutput = response.data.roadmap;
		} catch (error) {
			console.error('Error calling backend API:', error);
			errorMessage = 'Failed to generate roadmap. Please check console for details.';
		} finally {
			loading = false;
		}
	}
</script>

<main>
	<h1>MVP Roadmap Generator</h1>

	<Label for="idea">Idea Description</Label>
	<Input
		id="idea"
		bind:value={ideaDescription}
		placeholder="Enter your idea description here..."
		class="mb-4 w-full"
	/>

	<Button on:click={generateRoadmap} disabled={loading}>
		{#if loading}
			Generating Roadmap...
		{:else}
			Generate Roadmap
		{/if}
	</Button>

	{#if errorMessage}
		<p class="error">Error: {errorMessage}</p>
	{/if}

	{#if roadmapOutput}
		<h2>Roadmap Output:</h2>
		<pre>{JSON.stringify(roadmapOutput, null, 2)}</pre>
	{/if}

	{#if conversationId}
		<h2>Conversation ID:</h2>
		<pre>{conversationId}</pre>
	{/if}
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 20px;
	}

	.error {
		color: red;
		margin-top: 10px;
	}

	pre {
		background-color: #f4f4f4;
		padding: 10px;
		border: 1px solid #ddd;
		border-radius: 4px;
		overflow-x: auto; /* Enable horizontal scrolling for long output */
		white-space: pre-wrap; /* Wrap long lines */
		margin-top: 20px;
		font-size: 0.9em;
	}
</style>
