<!-- src/lib/components/Chat.svelte -->
<script>
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Button } from '$lib/components/ui/button';
	import LoadingIndicator from './LoadingIndicator.svelte';
	import ErrorMessage from './ErrorMessage.svelte';
	import RoadmapDisplay from './RoadmapDisplay.svelte';

	let message = '';
	let loading = false;
	let errorMessage = '';
	let chatHistory = [];
	let sessionId = null;
	let currentRoadmap = null;

	function isRoadmapRequest(msg) {
		const roadmapTriggers = ['generate roadmap', 'create roadmap', 'build roadmap'];
		return roadmapTriggers.some((trigger) => msg.toLowerCase().includes(trigger));
	}

	async function sendMessage() {
		if (!message.trim()) return;

		loading = true;
		errorMessage = '';
		const isRoadmap = isRoadmapRequest(message);

		try {
			const response = await fetch('http://localhost:3000/chat', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					message,
					sessionId
				})
			});

			if (!response.ok) {
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			const data = await response.json();
			sessionId = data.sessionId;

			// Handle different response types
			if (data.type === 'roadmap') {
				currentRoadmap = data.response;
				chatHistory = [
					...chatHistory,
					{ role: 'user', content: message },
					{
						role: 'assistant',
						content: 'I have generated a roadmap based on your request. You can view it below.',
						isRoadmap: true
					}
				];
			} else {
				chatHistory = [
					...chatHistory,
					{ role: 'user', content: message },
					{ role: 'assistant', content: data.response }
				];
			}
			message = '';
		} catch (error) {
			console.error('Error calling chat API:', error);
			errorMessage = 'Failed to send message. Please try again later.';
		} finally {
			loading = false;
		}
	}
</script>

<div class="space-y-4">
	<div class="h-[400px] overflow-y-auto rounded-lg border bg-white p-4 dark:bg-gray-800">
		{#if chatHistory.length === 0}
			<p class="text-center text-gray-500">
				Start a conversation by sending a message. You can ask questions or request to generate a
				roadmap.
			</p>
		{:else}
			{#each chatHistory as message}
				<div class="mb-4 {message.role === 'user' ? 'text-right' : 'text-left'}">
					<div
						class="inline-block max-w-[80%] rounded-lg px-4 py-2 {message.role === 'user'
							? 'bg-blue-500 text-white'
							: 'bg-gray-100 dark:bg-gray-700'}"
					>
						{message.content}
					</div>
				</div>
			{/each}
		{/if}
	</div>

	{#if currentRoadmap}
		<div class="rounded-lg border bg-white p-4 dark:bg-gray-800">
			<RoadmapDisplay roadmap={currentRoadmap} />
		</div>
	{/if}

	{#if errorMessage}
		<ErrorMessage message={errorMessage} />
	{/if}

	<div class="flex gap-2">
		<Input
			bind:value={message}
			placeholder="Type your message or ask to generate a roadmap..."
			on:keydown={(e) => e.key === 'Enter' && !e.shiftKey && sendMessage()}
			disabled={loading}
		/>
		<Button on:click={sendMessage} disabled={loading} class="w-24">
			{#if loading}
				<LoadingIndicator />
			{:else}
				Send
			{/if}
		</Button>
	</div>
</div>
