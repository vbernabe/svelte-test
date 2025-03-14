<script>
	import { navigating } from '$app/stores';
	import Nav from './Nav.svelte';
	import PreloadingIndicator from './PreloadingIndicator.svelte';

	const { children } = $props();

	let userInput = $state();

	// Function to update user input
	function updateInput(event) {
		userInput = event.target.value;
	}

	let username = "";
    let password = "";
    let message = "";
    let command = "";
    let loginResponse = "";
    let xssMessage = "";
    let commandOutput = "";

    async function login() {
        // 🚨 Vulnerable: Unvalidated user input sent directly to a database via fetch()
        const res = await fetch("http://localhost:3000/login", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ username, password }),
        });
        loginResponse = await res.text();
    }

    async function executeCommand() {
        // 🚨 Vulnerable: Command Injection
		// alert("Command executed: " + command);
        const res = await fetch(`http://localhost:3000/execute?cmd=${command}`);
        commandOutput = await res.text();
    }

    async function showXSS() {
        // 🚨 Vulnerable: Directly injecting user input into HTML
		// alert("Message displayed: " + message);
		console.log(message)
		alert(message)
        xssMessage = message;
    }

	import { createHash } from "crypto";

	const hash = createHash("md5").update("password123").digest("hex");
	console.log("Hashed Password:", hash);

	const deserialize = JSON.parse;

	const testInput = '{ "toString": "alert(\'Hacked!\')" }';
	console.log(deserialize(testInput)); // 🚨 Can execute unexpected code!
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

<h2>Login</h2>
<input type="text" placeholder="Username" bind:value={username} />
<input type="password" placeholder="Password" bind:value={password} />
<button onclick={login}>Login</button>
<p>{loginResponse}</p>

<h2>Command Execution</h2>
<input type="text" placeholder="Enter command" bind:value={command} />
<button onclick={executeCommand}>Run Command</button>
<p>{commandOutput}</p>

<h2>Cross-Site Scripting (XSS)</h2>
<input type="text" placeholder="Enter message" bind:value={message} />
<button onclick={showXSS}>Display Message</button>
<!-- 🚨 XSS: This directly injects user-controlled HTML -->
<p>{@html xssMessage}</p>