<script lang="ts">
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-sdk";
  import { TezosToolkit } from "@taquito/taquito";
  import { MichelsonMap } from "@taquito/michelson-encoder";

  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);
  const network = NetworkType.GHOSTNET;
  const contractAddress = "KT1N1UUq55cav1nE97gyPnbB7kESVv2vPvBm";

  let wallet;
  let address;
  let balance;
  let statusMessage = "Mint an NFT";
  let buttonActive = false;

  const connectWallet = async () => {
    try {
      const newWallet = new BeaconWallet({
        name: "Simple NFT app tutorial",
        preferredNetwork: network,
      });
      await newWallet.requestPermissions({
        network: { type: network, rpcUrl },
      });
      address = await newWallet.getPKH();
      const balanceMutez = await Tezos.tz.getBalance(address);
      balance = balanceMutez.div(1000000).toFormat(2);
      buttonActive = true;
      wallet = newWallet;
    } catch (error) {
      console.error("Error connecting wallet:", error);
    }
  };

  const disconnectWallet = () => {
    wallet.client.clearActiveAccount();
    wallet = undefined;
    buttonActive = false;
  };

  const requestNFT = async () => {
    if (!buttonActive) {
      return;
    }
    buttonActive = false;
    statusMessage = "Minting NFT...";

    const currentDate = new Date().toISOString().split('T')[0]; 
    const randomNumber = Math.floor(Math.random() * 10000);  
    const metadata = new MichelsonMap();
    metadata.set('serialNumber', `${currentDate}-${randomNumber}`);

    try {
      Tezos.setWalletProvider(wallet);

      const contract = await Tezos.wallet.at(contractAddress);
      const op = await contract.methods.mint(address, metadata).send({
        gasLimit: 100000,
        fee: 100000,
        storageLimit: 1,
        amount: 0,
      });

      console.log(`Waiting for ${op.opHash} to be confirmed...`);
      const hash = await op.confirmation(3).then(() => op.opHash);
      console.log(`Operation injected: https://ghost.tzstats.com/${hash}`);
    } catch (error) {
      console.error("Error minting NFT:", error);
    } finally {
      statusMessage = "Mint another NFT";
      buttonActive = true;
    }
  };

</script>

<main>
  <h1>Simple NFT dApp</h1>

  <div class="card">
    {#if wallet}
      <p>The address of the connected wallet is {address}.</p>
      <p>Its balance in tez is {balance}.</p>
      <button on:click={disconnectWallet}> Disconnect wallet </button>
      <button on:click={requestNFT}>{statusMessage}</button>
    {:else}
      <button on:click={connectWallet}> Connect wallet </button>
    {/if}
  </div>
</main>

<style>
</style>
