<script lang="ts">
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-sdk";
  import { TezosToolkit } from "@taquito/taquito";

  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);
  const network = NetworkType.GHOSTNET;
  const contractAddress = "KT1Rm7iM4wBLcyYTS8Q1RmULw7fojQvPWi7g";

  let wallet;
  let address;
  let balance;
  let statusMessage = "Mint an NFT";
  let buttonActive = false;

  const connectWallet = async () => {
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
    Tezos.setWalletProvider(wallet);
    console.log("minting");
    buttonActive = false;
    statusMessage = "Minting NFT...";

    await Tezos.wallet.at(contractAddress)
      .then((contract) =>
        contract.methods.mintNFT().send({
          gasLimit: 100000,
          fee: 100000,
          storageLimit: 1,
          amount: 0,
        })
      )
      .then((op) => {
        console.log(`Waiting for ${op.opHash} to be confirmed...`);
        return op.confirmation(3).then(() => op.opHash);
      })
      .then((hash) => console.log(`Operation injected: https://ghost.tzstats.com/${hash}`))
      .catch((error) => console.log(`Error: ${JSON.stringify(error, null, 2)}`));

    statusMessage = "Mint an NFT";
    console.log("done");
    buttonActive = true;
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
