<script>
	import { tick } from 'svelte';

	/**
	 * @type {any[]}
	 */
	let response = [];
	/**
	 * @type {number | null | undefined}
	 */
	let intervalId = null;
	let isApiCallInProgress = false;

	const formatarDados = (
		/** @type {{ datasetsNames: any[]; tablesNames: any[]; status: any[]; boards: { count: any; names: any[]; }; transfers: { newItems: { count: any; message: any; }; updatedItems: { count: any; }; excludedItems: { count: any; }; }; }} */ dados
	) => {
		let formatado = '== Relatório de Dados ==\n\n';

		formatado += '-- Conjuntos de Dados --\n';
		formatado += 'Nomes: ' + dados.datasetsNames.join(', ') + '\n\n';

		formatado += '-- Tabelas --\n';
		formatado += 'Nomes: ' + dados.tablesNames.join(', ') + '\n\n';

		formatado += '-- Status --\n';
		dados.status.forEach(
			(/** @type {{ step: any; success: any; }} */ s, /** @type {number} */ index) => {
				formatado += `${index + 1}. ${s.step}: ${s.success ? '✅' : '❌'}\n`;
			}
		);
		formatado += '\n';

		formatado += '-- Quadros --\n';
		formatado += `Quantidade: ${dados.boards.count}\n`;
		formatado += 'Nomes: ' + dados.boards.names.join(', ') + '\n\n';

		formatado += '-- Transferências --\n';
		formatado += `Novos Itens: ${dados.transfers.newItems.count} (${dados.transfers.newItems.message})\n`;
		formatado += `Itens Atualizados: ${dados.transfers.updatedItems.count}\n`;
		formatado += `Itens Excluídos: ${dados.transfers.excludedItems.count}\n`;

		return formatado;
	};

	const apiCall = async () => {
		if (isApiCallInProgress) return;

		isApiCallInProgress = true;

		// Add a separator line if there is existing data
		if (response.length > 0) {
			response = [...response, '-----------------------------'];
		}

		try {
			const apiResponse = await fetch('http://localhost:3000/boards/logs');
			if (apiResponse.ok) {
				const dados = await apiResponse.json();
				const dadosFormatados = formatarDados(dados);
				response = [...response, `Registros recebidos:\n${dadosFormatados}`];
			} else {
				response = [...response, `Falha ao buscar registros: ${apiResponse.statusText}`];
			}
		} catch (erro) {
			response = [...response, `Erro ao buscar registros: ${erro}`];
		}

		// Wait for Svelte to update the DOM
		await tick();

		// Then smoothly scroll to the bottom
		const terminalContainer = document.getElementById('terminal-container');
		if (terminalContainer) {
			terminalContainer.scrollTo({ top: terminalContainer.scrollHeight, behavior: 'smooth' });
		}

		isApiCallInProgress = false;
	};

	const startSync = () => {
		// Clear existing interval if any
		if (intervalId) {
			clearInterval(intervalId);
		}

		// Run once immediately upon clicking
		apiCall();

		// Then set up the interval to run every minute
		intervalId = setInterval(apiCall, 5000);
	};
</script>

<svelte:head>
	<title>Terminal - Black Beans</title>
	<meta name="description" content="Terminal Black Beans para visualização de Logs" />
</svelte:head>

<div class="row terminal" id="terminal-container">
	{#each response as data}
		<pre>{data}</pre>
	{/each}
</div>

<div class="row">
	<button class="mt-2 rounded-3 bg-dark me-2" on:click={() => startSync()}>Sincronizar dados</button
	>
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
