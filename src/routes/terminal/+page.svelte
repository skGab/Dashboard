<script>
	/**
	 * @type {any[]}
	 */
	let response = [];

	const teste = async () => {
		try {
			const apiResponse = await fetch('http://localhost:3000/boards/logs');
			if (apiResponse.ok) {
				const data = await apiResponse.json();
				response = [...response, `Received logs: ${JSON.stringify(data, null, 2)}`];
			} else {
				response = [...response, `Failed to fetch logs: ${apiResponse.statusText}`];
			}
		} catch (error) {
			response = [...response, `Error fetching logs: ${error}`];
		}
	};
</script>

<svelte:head>
	<title>Terminal - Black Beans</title>
	<meta name="description" content="Terminal Black Beans para visualização de Logs" />
</svelte:head>

<div class="row terminal">
	{#each response as data}
		<pre>{data}</pre>
	{/each}
</div>

<div class="row">
	<button class="mt-2 rounded-3 bg-dark me-2" on:click={() => teste()}>Sincronizar dados</button>
	<button class="mt-2 rounded-3 bg-dark"> <a href="/">Voltar</a></button>
</div>

<style>
	button {
		color: #ff9900;
		max-width: fit-content;
	}

	a {
		color: #ff9900;
	}

	.terminal {
		background-color: black;
		color: #ff9900;
		font-family: 'Courier New', monospace;
		padding: 1rem;
		overflow-y: scroll;
		min-height: 90vh;
		max-height: 90vh;
		border-radius: 8px;

		transition: all ease-in 0.5s;
	}

	pre {
		overflow-x: hidden;
	}
</style>
