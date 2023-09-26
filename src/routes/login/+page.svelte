<script>
	import { signIn, signOut } from '@auth/sveltekit/client';
	import { page } from '$app/stores';
	import logo from '$lib/assets/logo.png';
	import Google from '$lib/components/icon/Google.svelte';

	const handleGoogleSignIn = () => {
		signIn('google', { callbackUrl: '/terminal' });
	};

	const handleSignOut = () => {
		signOut();
	};
</script>

<div class="container py-5 d-flex flex-column justify-content-between">
	{#if !$page.data.session}
		<img src={logo} width="150" alt="auth.js logo" class="logo mb-3" />
		<div>
			<h1 class="welcome">Dashboard Black Beans</h1>
			<p class="text mb-4">Login para acesso ao terminal</p>
		</div>

		<button on:click={handleGoogleSignIn}>
			<Google /> Continuar com Google
		</button>
	{/if}

	{#if $page.data.session}
		<div class="signout-container">
			<img src={logo} alt="auth.js logo" class="logo" />

			<button on:click={handleSignOut}>Sign out</button>
		</div>
	{/if}
</div>

<style>
	.container {
		padding: 1rem;
		margin: 0 auto;
		border-radius: 0.375rem;
		width: 22rem;
		height: 30rem;
		box-shadow: 0px 2px 16px 0px rgba(0, 0, 0, 0.25);
	}

	.logo {
		display: block;
		margin: 0 auto;
	}

	.welcome {
		font-size: 1.5rem;
		line-height: 2rem;
		text-align: center;
	}

	.text {
		text-align: center;
	}

	button {
		background: none;
		margin-top: 1.5rem;
		display: flex;
		width: 100%;
		padding: 0.75rem;
		gap: 1rem;
		justify-content: center;
		align-items: center;
		border: 1px solid #ff9900;
		border-radius: 0.125rem;
		transition-timing-function: all 225ms cubic-bezier(0.4, 0, 0.2, 1);
	}

	.signout-container {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		height: 100%;
	}

	.signout-container img {
		flex: 1;
		object-fit: contain;
		width: 75%;
		justify-self: center;
	}
</style>
