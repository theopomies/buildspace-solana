<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  const connectWallet = async () => {
    const { solana } = window;
    const response = await solana.connect();
    dispatch("connect", response.publicKey.toString());
  };
</script>

<button class="cta-button connect-wallet-button" on:click={connectWallet}>
  Connect to Wallet
</button>

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
  }

  .connect-wallet-button {
    background: transparent;
    border: 2px solid white;
    background-size: 200% 200%;
  }

  .connect-wallet-button:hover {
    background: white;
    color: var(--primary);
  }
</style>
