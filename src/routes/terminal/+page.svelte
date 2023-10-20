<script>
	import { tick } from 'svelte';

	/**
	 * @type {string | any[]}
	 */
	let response = [];
	let intervalId = null;
	let isApiCallInProgress = false;
	let progress = 0;

	const formatarDados = (
		/** @type {{ datasetJobStatus?: { avaliableDatasets?: any[]; mondayWorkspaces?: { names?: any[]; count?: string; status?: string; }; }; tableJobStatus?: { avaliableTables?: any[]; newTables?: string; boardDto?: { count?: any; names?: any[]; }; }; itemsJobStatus?: { operationStatus?: any[]; }; }} */ dados
	) => {
		let formatado = '== Relatório de Dados ==\n\n';

		// Datasets
		formatado += '-- Datasets --\n';
		formatado +=
			'Nomes: ' + (dados?.datasetJobStatus?.mondayWorkspaces?.names?.join(', ') || 'N/A') + '\n';
		formatado +=
			'Quantidade: ' + (dados?.datasetJobStatus?.mondayWorkspaces?.count || 'N/A') + '\n';
		formatado += 'Status: ' + (dados?.datasetJobStatus?.mondayWorkspaces?.status || 'N/A') + '\n\n';

		// Tables
		formatado += '-- Tabelas --\n';
		formatado +=
			'Disponíveis: ' + (dados?.tableJobStatus?.avaliableTables?.join(', ') || 'N/A') + '\n';
		formatado += 'Novas: ' + (dados?.tableJobStatus?.newTables || 'N/A') + '\n\n';

		// Items
		formatado += '-- Itens --\n';
		dados?.itemsJobStatus?.operationStatus?.forEach(
			(
				/** @type {{ table?: any; count?: any; status?: any; }} */ operation,
				/** @type {number} */ index
			) => {
				formatado += `${index + 1}. Tabela: ${operation?.table || 'N/A'}, Itens: ${
					operation?.count || 'N/A'
				}, Status: ${operation?.status || 'N/A'}\n`;
			}
		);

		return formatado;
	};

	let isLoading = false; // New loading state variable

	const totalTimeMs = 240000; // 2 minutes in milliseconds
	const intervalTimeMs = 50; // Interval time in milliseconds
	const steps = totalTimeMs / intervalTimeMs; // Total number of steps
	const progressIncrement = 100 / steps; // Progress to be incremented in each step

	const apiCall = async () => {
		if (isApiCallInProgress) return;

		isApiCallInProgress = true;
		isLoading = true; // Set to true before making the API call

		// Add a separator line if there is existing data
		if (response.length > 0) {
			response = [...response, '-----------------------------'];
		}

		progress = 0;
		intervalId = setInterval(() => {
			if (progress < 100) {
				progress += progressIncrement;
			}
		}, intervalTimeMs);

		try {
			const apiResponse = await fetch('http://localhost:3000/integration/logs');
			if (apiResponse.ok) {
				const dados = await apiResponse.json();
				const dadosFormatados = formatarDados(dados);
				response = [...response, `Registros recebidos:\n${dadosFormatados}`];
			} else {
				response = [...response, `Falha ao buscar registros: ${apiResponse.statusText}`];
			}
		} catch (erro) {
			if (erro instanceof TypeError) {
				response = [...response, 'TypeError: Some properties are not available in the data'];
			} else {
				response = [...response, `Erro ao buscar registros: ${erro}`];
			}
		}

		// Wait for Svelte to update the DOM
		await tick();

		// Then smoothly scroll to the bottom
		const terminalContainer = document.getElementById('terminal-container');
		if (terminalContainer) {
			terminalContainer.scrollTo({ top: terminalContainer.scrollHeight, behavior: 'smooth' });
		}

		isLoading = false; // Set back to false once API call is done
		isApiCallInProgress = false;

		// Clear interval and set progress to 100 after API call
		clearInterval(intervalId);
		progress = 100;
	};

	// const startSync = () => {
	// 	// Clear existing interval if any
	// 	if (intervalId) {
	// 		clearInterval(intervalId);
	// 	}

	// 	// Run once immediately upon clicking
	// 	apiCall();

	// 	// Then set up the interval to run every minute
	// 	intervalId = setInterval(apiCall, 5000);
	// };
</script>

<svelte:head>
	<title>Terminal - Black Beans</title>
	<meta name="description" content="Terminal Black Beans para visualização de Logs" />
</svelte:head>

<div class="row d-block terminal" id="terminal-container">
	{#each response as data}
		<pre>{data}</pre>
	{/each}
</div>

<div class="row mt-2 align-items-center">
	<!-- BUTTONS -->
	{#if !isLoading}
		<button class="rounded-3 bg-dark me-2" on:click={apiCall}>Sincronizar dados</button>
		<button class="rounded-3 bg-dark"> <a href="/">Voltar</a></button>
	{/if}
	<!-- LOADING BAR -->

	{#if isLoading}
		<div class="loading-bar mt-3 ms-4 fw-bold" style="width: {progress}%">Carregando...</div>
	{/if}
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
		height: fit-content;
	}

	.loading-bar {
		height: 4px;
		background: linear-gradient(90deg, #ff9900 25%, transparent 25%),
			linear-gradient(90deg, #ff9900 25%, transparent 25%),
			linear-gradient(90deg, transparent 75%, #ff9900 75%),
			linear-gradient(90deg, transparent 75%, #ff9900 75%);
		background-size: 50px 4px;
		background-position: 0 0, 25px 0, 25px -4px, 0px 4px;
		animation: loading-animation 1s linear infinite;
		/* position: absolute;
		bottom: 5%; */
	}

	@keyframes loading-animation {
		0% {
			background-position: 0 0, 25px 0, 25px -4px, 0px 4px;
		}
		100% {
			background-position: 50px 0, 75px 0, 75px -4px, 50px 4px;
		}
	}
</style>
