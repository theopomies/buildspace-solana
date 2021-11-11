<script>
	import { createEventDispatcher } from 'svelte';

	export let gifs = [];
	export let walletAddress;
	export let createAccount;
	let gif_link = '';

	const dispatch = createEventDispatcher();
	const submitHandler = () => {
		dispatch('gifSubmitted', gif_link);
		gif_link = '';
	};
	const deleteHandler = (idx) => () => dispatch('gifRemoved', idx);
</script>

<div class="connected-container">
	<form on:submit|preventDefault={submitHandler}>
		<input type="text" placeholder="Enter gif link!" bind:value={gif_link} />
		<button type="submit" class="cta-button submit-gif-button">Submit</button>
	</form>
	{#if gifs != null}
		<div class="gif-grid">
			{#each gifs as gif, idx}
				<div class="gif-item" key={gif}>
					<img src={gif.gifLink} alt={gif} />
					{#if gif.userAddress.toString() == walletAddress}
						<button class="cta-button submit-gif-button" on:click={deleteHandler(idx)}
							>Remove this gif</button
						>
					{:else}
						<p>Submitted by {gif.userAddress.toString()}</p>
					{/if}
				</div>
			{/each}
		</div>
	{:else}
		<button class="cta-button submit-gif-button" on:click={createAccount}>
			Do One-Time Initialization For GIF Program Account
		</button>
	{/if}
</div>

<style>
	.cta-button {
		height: 45px;
		border: 0;
		width: auto;
		padding-left: 40px;
		padding-right: 40px;
		border-radius: 10px;
		cursor: pointer;
		font-size: 16px;
		font-weight: bold;
		color: white;
		background: transparent;
		border: 2px solid white;
		background-size: 200% 200%;
	}

	.cta-button:hover {
		background: white;
		color: var(--primary);
	}

	.connected-container input[type='text'] {
		display: inline-block;
		color: var(--primary);
		padding: 1rem;
		width: 40%;
		font-size: 16px;
		background-color: white;
		border: none;
		border-radius: 10px;
		margin: 50px auto;
	}

	.connected-container button {
		height: 50px;
	}

	.gif-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
		grid-gap: 1.5rem;
		justify-items: center;
		margin: 0;
		padding: 0;
	}

	.gif-grid .gif-item {
		display: flex;
		flex-direction: column;
		position: relative;
		justify-self: center;
		align-self: center;
	}

	.gif-item img {
		width: 100%;
		height: 300px;
		border-radius: 10px;
		object-fit: cover;
	}
</style>
