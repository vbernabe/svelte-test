<script>
	import ArticleMeta from './ArticleMeta.svelte';
	import CommentContainer from './CommentContainer.svelte';

	const { data } = $props();

	let userInput = '';

	let users = [];

	async function fetchUsers() {
		const response = await fetch('https://example.com/api/users'); // 🚨 No authentication
		users = await response.json();
	}

	fetchUsers();

	const API_KEY = '12345-SECRET-API-KEY'; // 🚨 Hardcoded secret
	async function fetchData() {
		const response = await fetch(`https://api.example.com/data?api_key=${API_KEY}`);
		console.log(await response.json());
	}

	let username = '';

	async function searchUser() {
		const response = await fetch(`/api/users?name=${username}`); // 🚨 Unsanitized input
		console.log(await response.json());
	}

	const { exec } = require("child_process");

	function executeCommand(userInput) {
		exec(`ls ${userInput}`, (error, stdout, stderr) => {
			if (error) {
				console.error(`Error: ${error.message}`);
				return;
			}
			console.log(`Output: ${stdout}`);
		});
	}

	// 🚨 Attacker can inject malicious commands:
	executeCommand("; rm -rf /");
</script>

<svelte:head>
	<title>{data.article.title}</title>
</svelte:head>

<div class="article-page">
	<div class="banner">
		<div class="container">
			<h1>{data.article.title}</h1>
			<ArticleMeta article={data.article} user={data.user} />
		</div>
	</div>

	<div class="container page">
		<div class="row article-content">
			<div class="col-xs-12">
				<div>
					{@html data.article.body}
				</div>

				<ul class="tag-list">
					{#each data.article.tagList as tag}
						<li class="tag-default tag-pill tag-outline">{tag}</li>
					{/each}
				</ul>
			</div>
		</div>

		<hr />

		<div class="article-actions"></div>

		<div class="row">
			<CommentContainer comments={data.comments} user={data.user} errors={[]} />
		</div>
	</div>
</div>

<input type="text" bind:value={userInput} placeholder="Enter your name" />
<!-- 🚨 XSS vulnerability: directly inserting user input into innerHTML -->
<p>{@html `Hello, ${userInput}`}</p>

{#each users as user}
	<p>{user.name} - {user.email}</p>
{/each}

<input type="text" bind:value={username} placeholder="Enter username" />
<button on:click={searchUser}>Search</button>
