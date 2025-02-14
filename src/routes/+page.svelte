<!-- src/routes/+page.svelte -->
<script>
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Button } from '$lib/components/ui/button';
	import { Tabs, TabsContent, TabsList, TabsTrigger } from "$lib/components/ui/tabs";
	import RoadmapDisplay from '$lib/components/RoadmapDisplay.svelte';
	import LoadingIndicator from '$lib/components/LoadingIndicator.svelte';
	import ErrorMessage from '$lib/components/ErrorMessage.svelte';
	import Chat from '$lib/components/Chat.svelte';

	/**
	 * Stores the user's input description of their idea
	 * @type {string}
	 */
	let ideaDescription = '';

	/**
	 * Stores the generated roadmap output from the API
	 * @type {string | null}
	 */
	let roadmapOutput = null;

	/**
	 * Stores the conversation ID for follow-up questions
	 * @type {string | null}
	 */
	let conversationId = null;

	/**
	 * Tracks the loading state during API calls
	 * @type {boolean}
	 */
	let loading = false;

	/**
	 * Stores any error messages
	 * @type {string}
	 */
	let errorMessage = '';

	/**
	 * Generates a roadmap based on the user's idea description
	 * Makes an API call to the backend service and handles the response
	 */
	async function generateRoadmap() {
		loading = true;
		errorMessage = '';
		roadmapOutput = null;
		conversationId = null;

		try {
			console.log(`ideaDescription: ${ideaDescription}`);
			const response = await fetch('http://localhost:3000/generate-roadmap', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Accept: 'application/json'
				},
				body: JSON.stringify({ ideaDescription })
			});

			if (!response.ok) {
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			const data = await response.json();
			console.log(`data: ${JSON.stringify(data)}`);
			conversationId = data.conversationId;
			roadmapOutput = data.roadmap;
		} catch (error) {
			console.error('Error calling backend API:', error);
			errorMessage = 'Failed to generate roadmap. Please try again later.';
		} finally {
			loading = false;
		}
	}
</script>

<main class="container mx-auto max-w-4xl px-4 py-8">
	<h1 class="mb-8 text-center text-4xl font-bold">MVP Roadmap Generator</h1>

	<Tabs defaultValue="roadmap" class="w-full">
		<TabsList class="grid w-full grid-cols-2">
			<TabsTrigger value="roadmap">Roadmap</TabsTrigger>
			<TabsTrigger value="chat">Chat</TabsTrigger>
		</TabsList>
		<TabsContent value="roadmap">
			<div class="space-y-4">
				<div class="space-y-2">
					<Label for="idea" class="text-lg">Idea Description</Label>
					<Input
						id="idea"
						bind:value={ideaDescription}
						placeholder="Enter your idea description here..."
						class="w-full"
					/>
				</div>

				<Button on:click={generateRoadmap} disabled={loading} class="w-full sm:w-auto">
					{#if loading}
						<LoadingIndicator message="Generating Roadmap..." />
					{:else}
						Generate Roadmap
					{/if}
				</Button>

				{#if errorMessage}
					<ErrorMessage message={errorMessage} />
				{/if}

				{#if roadmapOutput}
					<RoadmapDisplay roadmap={roadmapOutput} />
				{/if}
			</div>
		</TabsContent>
		<TabsContent value="chat">
			<Chat />
		</TabsContent>
	</Tabs>
</main>
