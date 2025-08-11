<script>
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-types";
  import { TezosToolkit } from "@taquito/taquito";

  // const rpcUrl = "http://localhost:46083/tezlink";
  //const rpcUrl = "https://rpc.tzkt.io/ghostnet";

  const rpcUrl = "http://localhost:1046";
  const Tezos = new TezosToolkit(rpcUrl);
  // tezlink
  const counterAddress = "KT1Q36KWPSba7dHsH5E4ZsQHehrChc51e19d";
  // ghostnet
  //const counterAddress = "KT1NDyxKBRVrfPRvEDEWVKMF17TjwcSiPdcc";

  let wallet;
  let address;
  let balance;
  let bankBalance;
  let counter = 0;

  let incrementAmount = 1;
  let incrementButtonActive = false;
  let incrementButtonLabel = "Increment";

  let decrementAmount = 1;
  let decrementButtonActive = false;
  let decrementButtonLabel = "Decrement";

  const connectWallet = async () => {
    const newWallet = new BeaconWallet({
      name: "Simple dApp tutorial",
      network: {
        type: NetworkType.CUSTOM,
      },
    });
    await newWallet.requestPermissions();
    address = await newWallet.getPKH();
    await getWalletBalance(address);
    wallet = newWallet;
    incrementButtonActive = true;
    decrementButtonActive = true;
    await getCounter();
  };

  const disconnectWallet = () => {
    wallet.client.clearActiveAccount();
    wallet = undefined;
  };

  const getWalletBalance = async (walletAddress) => {
    const balanceMutez = await Tezos.tz.getBalance(walletAddress);
    balance = balanceMutez.div(1000000).toFormat(2);
  };

  const getCounter = async () => {
    const contract = await Tezos.wallet.at(counterAddress);
        contract.storage().then((store) => counter = store.toNumber())
      }
  const decrement = async () => {
    decrementButtonActive = false;
    decrementButtonLabel = "Decrementing...";

    Tezos.setWalletProvider(wallet);
    const contract = await Tezos.wallet.at(counterAddress);
    const operation = await contract.methods
      .decrement(1).send();

    console.log(`Waiting for ${operation.opHash} to be confirmed...`);

    await operation.confirmation(2);

    console.log(
      `Operation injected: https://ghost.tzstats.com/${operation.opHash}`
    );

    await getWalletBalance(address);
    await getCounter();
    decrementButtonActive = true;
    decrementButtonLabel = "Decrement";
  };

  const increment = async () => {
    incrementButtonActive = false;
    incrementButtonLabel = "Incrementing...";

    Tezos.setWalletProvider(wallet);
    const contract = await Tezos.wallet.at(counterAddress);
    const operation = await contract.methods
      .increment(1).send()
      ;

    console.log(`Waiting for ${operation.opHash} to be confirmed...`);

    await operation.confirmation(2);

    console.log(
      `Operation injected: https://ghost.tzstats.com/${operation.opHash}`
    );
    await getWalletBalance(address);
    await getCounter();
    incrementButtonActive = true;
    incrementButtonLabel = "Increment";
  };
</script>

<main>
  <h1>Tezos bank dApp</h1>

  <div class="card">
    {#if wallet}
      <p>The address of the connected wallet is {address}.</p>
      <p>Its balance in tez is {balance}.</p>
      <p>Counter is {counter}.</p>
      <p>
        To get tez, go to <a
          href="https://faucet.ghostnet.teztnets.xyz/"
          target="_blank"
        >
          https://faucet.ghostnet.teztnets.xyz/
        </a>.
      </p>
      <p>
        <button on:click={increment} disabled={!incrementButtonActive}>
          {incrementButtonLabel}
        </button>
        <button on:click={decrement} disabled={!decrementButtonActive}>
          {decrementButtonLabel}
        </button>
      </p>
      <p>
        <button on:click={disconnectWallet}> Disconnect wallet </button>
      </p>
    {:else}
      <button on:click={connectWallet}> Connect wallet </button>
    {/if}
  </div>
</main>

<style>
</style>
