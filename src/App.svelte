<script lang="ts">
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-sdk";
  import { TezosToolkit } from "@taquito/taquito";

  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);
  const network = NetworkType.GHOSTNET;

  let wallet;
  let address;

  const connectWallet = async () => {
    const newWallet = new BeaconWallet({
      name: "Simple NFT app tutorial",
      preferredNetwork: network,
    });
    await newWallet.requestPermissions({
      network: { type: network, rpcUrl },
    });
    address = await newWallet.getPKH();
    wallet = newWallet;
  };

  const disconnectWallet = () => {
    wallet.client.clearActiveAccount();
    wallet = undefined;
  };
</script>

<main>
  <h1>Simple NFT dApp</h1>

  <div class="card">
    {#if wallet}
      <p>The address of the connected wallet is {address}.</p>
      <button on:click={disconnectWallet}>
        Disconnect wallet
      </button>
    {:else}
      <button on:click={connectWallet}>
        Connect wallet
      </button>
    {/if}
  </div>
</main>

<style>
</style>
