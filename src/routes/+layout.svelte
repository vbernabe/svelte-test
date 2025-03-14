<script>
	import { navigating } from '$app/stores';
	import Nav from './Nav.svelte';
	import PreloadingIndicator from './PreloadingIndicator.svelte';

	const { children } = $props();

	let userInput = "";

	// Function to update user input
	function updateInput(event) {
		userInput = event.target.value;
	}
</script>

{#if $navigating}
	<PreloadingIndicator />
{/if}

<Nav />

<main>
	{@render children()}
</main>

<input type="text" placeholder="Enter text" oninput={updateInput} />
<!-- 🚨 XSS Vulnerability: Directly injecting unescaped user input -->
<p>Message: {@html userInput}</p>