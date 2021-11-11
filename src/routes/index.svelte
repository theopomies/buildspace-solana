<script context="module">
	export const prerender = true;
</script>

<script>
	import ConnectedContainer from '$lib/ConnectedContainer.svelte';
	import ConnectWalletButton from '$lib/ConnectWalletButton.svelte';
	import { onMount } from 'svelte';

	import { Connection, PublicKey, clusterApiUrl } from '@solana/web3.js';
	import { Program, Provider, web3 } from '@project-serum/anchor';
	import idl from '$lib/idl.json';
	const { SystemProgram } = web3;
	import kp from '$lib/keypair.json';

	const TITLE = '🦑 SquidGifs';
	const TEST_GIFS = [
		'https://i.giphy.com/media/eIG0HfouRQJQr1wBzz/giphy.webp',
		'https://media3.giphy.com/media/L71a8LW2UrKwPaWNYM/giphy.gif?cid=ecf05e47rr9qizx2msjucl1xyvuu47d7kf25tqt2lvo024uo&rid=giphy.gif&ct=g',
		'https://media4.giphy.com/media/AeFmQjHMtEySooOc8K/giphy.gif?cid=ecf05e47qdzhdma2y3ugn32lkgi972z9mpfzocjj6z1ro4ec&rid=giphy.gif&ct=g',
		'https://i.giphy.com/media/PAqjdPkJLDsmBRSYUp/giphy.webp'
	];

	const arr = Object.values(kp._keypair.secretKey);
	const secret = new Uint8Array(arr);
	const baseAccount = web3.Keypair.fromSecretKey(secret);
	const programID = new PublicKey(idl.metadata.address);
	const network = clusterApiUrl('devnet');
	const opts = {
		preflightCommitment: 'processed'
	};

	const getProvider = () => {
		const connection = new Connection(network, opts.preflightCommitment);
		const provider = new Provider(connection, window.solana, opts.preflightCommitment);
		return provider;
	};

	const getGifList = async () => {
		try {
			const provider = getProvider();
			console.log(idl, programID, provider);
			const program = new Program(idl, programID, provider);
			console.log('done');
			const account = await program.account.baseAccount.fetch(baseAccount.publicKey);

			console.log('Got the account', account);
			return account.gifList;
		} catch (error) {
			console.log('Error in getGifs: ', error);
			return null;
		}
	};

	const checkIfWalletIsConnected = async () => {
		try {
			const { solana } = window;

			if (solana) {
				if (solana.isPhantom) {
					console.log('Phantom wallet found!');
					const response = await solana.connect({ onlyIfTrusted: true });
					address = response.publicKey.toString();
					console.log('Connected with Public Key:', address);
				}
			} else {
				alert('Solana object not found! Get a Phantom Wallet 👻');
			}
		} catch (e) {
			console.error(e);
		}
	};

	const createGifAccount = async () => {
		try {
			const provider = getProvider();
			console.log(idl, programID, provider);
			const program = new Program(idl, programID, provider);
			console.log('ping');
			await program.rpc.initialize({
				accounts: {
					baseAccount: baseAccount.publicKey,
					user: provider.wallet.publicKey,
					systemProgram: SystemProgram.programId
				},
				signers: [baseAccount]
			});
			console.log('Created a new BaseAccount w/ address:', baseAccount.publicKey.toString());
			gifs = await getGifList();
		} catch (error) {
			console.log('Error creating BaseAccount account:', error);
		}
	};
	const sendGif = async (value) => {
		if (value.length === 0) {
			console.log('No gif link given!');
			return;
		}
		console.log('Gif link:', value);
		try {
			const provider = getProvider();
			const program = new Program(idl, programID, provider);

			await program.rpc.addGif(value, {
				accounts: {
					baseAccount: baseAccount.publicKey,
					user: provider.wallet.publicKey
				}
			});
			console.log('GIF sucesfully sent to program', value);

			gifs = await getGifList();
		} catch (error) {
			console.log('Error sending GIF:', error);
		}
	};

	onMount(async () => {
		const { Buffer } = await import('buffer');
		window.Buffer = Buffer;
		checkIfWalletIsConnected;
	});

	let address = null;

	let gifs = null;
	$: {
		if (address != null) {
			console.log('Fetching GIF list...');
			getGifList().then((res) => (gifs = res));
		}
	}
</script>

<svelte:head>
	<title>{TITLE}</title>
</svelte:head>

<main>
	<div class="App">
		<div class={address == null ? 'container' : 'auth-container'}>
			<div class="header-container">
				<p class="header">{TITLE}</p>
				<p class="sub-text">Store your favorite SquidGifs on Solana!</p>
				<br />
				{#if address == null}
					<ConnectWalletButton on:connect={(e) => (address = e.detail)} />
				{:else}
					<br />
					<p>Connected as {address}</p>
					<br />
					<ConnectedContainer
						{gifs}
						createAccount={createGifAccount}
						on:gifSubmitted={(event) => {sendGif(event.detail)}}
					/>
				{/if}
			</div>
			<div class="footer-container">
				<a
					class="footer-text"
					href="https://twitter.com/theopomies"
					target="_blank"
					rel="noreferrer">Built by @theopomies</a
				>
			</div>
		</div>
	</div>
</main>

<style>
	.App {
		height: 100vh;
		background-color: var(--primary);
		overflow: scroll;
		text-align: center;
	}

	.container {
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		padding: 0 30px 0 30px;
	}
	/* 
	.authed-container {
		height: 100%;
		display: flex;
		flex-direction: column;
		padding: 30px;
	} */

	.header {
		margin: 0;
		font-size: 50px;
		font-weight: bold;
		color: white;
	}

	.sub-text {
		font-size: 25px;
		color: white;
	}

	.footer-container {
		display: flex;
		justify-content: center;
		align-items: center;
		position: absolute;
		width: 100%;
		bottom: 0;
		left: 0;
		padding-bottom: 45px;
	}

	.footer-text {
		color: white;
		font-size: 16px;
		font-weight: bold;
	}
</style>
