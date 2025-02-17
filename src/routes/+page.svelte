<!-- src/routes/+page.svelte -->
<script>
	import { Label } from '$lib/components/ui/label';
	import { Input } from '$lib/components/ui/input';
	import { Button } from '$lib/components/ui/button';
	import { Tabs, TabsContent, TabsList, TabsTrigger } from '$lib/components/ui/tabs';
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

	// Mock data for development
	const mockData = {
		roadmap: {
			idea: 'I want to create a thermo bottle',
			initialClarificationQuestions:
				"1. What key problem are you solving with this thermo bottle that isn't adequately addressed by existing solutions? (e.g., longer temperature retention, specific size/portability needs, unique material for specific beverages)\n2.  Who is your ideal customer for this thermo bottle?  (e.g., outdoor enthusiasts, office workers, students)\n",
			marketOverview: 'SSSSUP',
			marketOverviewGroundingMetadata: null,
			mvpEpics:
				"Assuming a focus on a thermo bottle differentiating itself through superior temperature retention and targeting outdoor enthusiasts (as these were not explicitly defined in previous turns, but are plausible assumptions for this example):\n\n1. **Core Temperature Performance:** This epic would encompass all features related to maximizing insulation and temperature retention.  This includes material selection,  manufacturing processes, and rigorous testing/validation.  The MVP should deliver demonstrably superior performance compared to mainstream competitors.\n\n2. **Durability and Portability:** For outdoor enthusiasts, ruggedness is key. This epic covers features like impact resistance, leak-proof design,  convenient carrying mechanisms, and potentially specialized attachments for backpacks or other gear.  The MVP should be demonstrably durable and easy to carry in outdoor settings.\n\n3. **Easy Cleaning and Maintenance:**  Hygiene and ease of use are important. This epic covers features related to cleaning (wide mouth, dishwasher safe components),  disassembly (if applicable), and long-term maintenance to preserve the bottle's performance.  The MVP should be demonstrably easy to clean and maintain.\n",
			taskBreakdown:
				'Epic: **Core Temperature Performance**\n\n* **Task 1:** Research and select optimal insulation materials (vacuum, aerogel, etc.).  *Complexity: Medium*\n* **Task 2:** Design and prototype inner and outer bottle construction for maximum insulation. *Complexity: High*\n* **Task 3:** Develop testing procedures to measure temperature retention over time. *Complexity: Medium*\n* **Task 4:** Conduct temperature retention tests on prototypes with various liquids (hot/cold). *Complexity: Medium*\n* **Task 5:** Analyze test results and iterate on design/materials to optimize performance. *Complexity: Medium*\n* **Task 6:** Finalize material selection and bottle construction for MVP production. *Complexity: Low*\n',
			conversationHistory: [
				{
					role: 'user',
					parts:
						'You are an expert product roadmap generator. A user has provided an idea: "I want to create a thermo bottle". Ask 2-3 concise questions to better understand their idea and its target users. Focus on clarifying the core functionality and target audience for an MVP.'
				},
				{
					role: 'model',
					parts:
						"1. What key problem are you solving with this thermo bottle that isn't adequately addressed by existing solutions? (e.g., longer temperature retention, specific size/portability needs, unique material for specific beverages)\n2.  Who is your ideal customer for this thermo bottle?  (e.g., outdoor enthusiasts, office workers, students)\n"
				},
				{
					role: 'user',
					parts:
						'Based on our discussion about "I want to create a thermo bottle", provide a brief market overview including potential market size, key competitors, and current trends. Use web search to ground your response in recent information. Keep it concise for an MVP roadmap context.'
				},
				{
					role: 'model',
					parts: 'Error in chat. Please check the logs.'
				},
				{
					role: 'user',
					parts:
						'Given our previous discussion about the market and the idea "I want to create a thermo bottle", suggest 2-3 high-level epics (major feature areas) for the MVP. Focus on core functionality that addresses the market needs we identified.'
				},
				{
					role: 'model',
					parts:
						"Assuming a focus on a thermo bottle differentiating itself through superior temperature retention and targeting outdoor enthusiasts (as these were not explicitly defined in previous turns, but are plausible assumptions for this example):\n\n1. **Core Temperature Performance:** This epic would encompass all features related to maximizing insulation and temperature retention.  This includes material selection,  manufacturing processes, and rigorous testing/validation.  The MVP should deliver demonstrably superior performance compared to mainstream competitors.\n\n2. **Durability and Portability:** For outdoor enthusiasts, ruggedness is key. This epic covers features like impact resistance, leak-proof design,  convenient carrying mechanisms, and potentially specialized attachments for backpacks or other gear.  The MVP should be demonstrably durable and easy to carry in outdoor settings.\n\n3. **Easy Cleaning and Maintenance:**  Hygiene and ease of use are important. This epic covers features related to cleaning (wide mouth, dishwasher safe components),  disassembly (if applicable), and long-term maintenance to preserve the bottle's performance.  The MVP should be demonstrably easy to clean and maintain.\n"
				},
				{
					role: 'user',
					parts:
						'Based on all our previous discussion about the market, idea, and epics, break down the first epic into specific, actionable tasks. Include estimated complexity (Low/Medium/High) for each task.'
				},
				{
					role: 'model',
					parts:
						'Epic: **Core Temperature Performance**\n\n* **Task 1:** Research and select optimal insulation materials (vacuum, aerogel, etc.).  *Complexity: Medium*\n* **Task 2:** Design and prototype inner and outer bottle construction for maximum insulation. *Complexity: High*\n* **Task 3:** Develop testing procedures to measure temperature retention over time. *Complexity: Medium*\n* **Task 4:** Conduct temperature retention tests on prototypes with various liquids (hot/cold). *Complexity: Medium*\n* **Task 5:** Analyze test results and iterate on design/materials to optimize performance. *Complexity: Medium*\n* **Task 6:** Finalize material selection and bottle construction for MVP production. *Complexity: Low*\n'
				}
			]
		}
	};

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
			// console.log(`ideaDescription: ${ideaDescription}`);
			// const response = await fetch('http://localhost:3000/generate-roadmap', {
			// 	method: 'POST',
			// 	headers: {
			// 		'Content-Type': 'application/json',
			// 		Accept: 'application/json'
			// 	},
			// 	body: JSON.stringify({ ideaDescription })
			// });

			// if (!response.ok) {
			// 	throw new Error(`HTTP error! status: ${response.status}`);
			// }

			// const data = await response.json();
			// console.log(`data: ${JSON.stringify(data)}`);
			// conversationId = mocked.conversationId;
			// roadmapOutput = mocked.roadmap;
			await new Promise((resolve) => setTimeout(resolve, 1000));
			roadmapOutput = mockData.roadmap;
			console.log(`roadmapOutput: ${JSON.stringify(roadmapOutput)}`);
		} catch (error) {
			console.error('Error:', error);
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
					<RoadmapDisplay {roadmapOutput} />
				{/if}
			</div>
		</TabsContent>
		<TabsContent value="chat">
			<Chat />
		</TabsContent>
	</Tabs>
</main>
