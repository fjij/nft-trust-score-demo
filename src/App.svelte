<script lang="ts">
  import { getCollectionDataset, computeTrustScore } from "nft-trust-score";
  import type { TrustScore } from "nft-trust-score";
  import { ethers } from "ethers";

  let collectionAddressA = "";
  let collectionAddressB = "";
  let formError = "";
  let trustScorePromise: Promise<TrustScore>;

  async function getTrustScore(addressA: string, addressB: string) {
    const [datasetA, datasetB] = await Promise.all([
      getCollectionDataset(addressA, true),
      getCollectionDataset(addressB, true),
    ]);
    return computeTrustScore(datasetA, datasetB);
  }

  function handleOnSubmit() {
    if (!ethers.utils.isAddress(collectionAddressA)) {
      formError = `Invalid address: ${collectionAddressA}`;
      return;
    }
    if (!ethers.utils.isAddress(collectionAddressA)) {
      formError = `Invalid address: ${collectionAddressB}`;
      return;
    }
    trustScorePromise = getTrustScore(collectionAddressA, collectionAddressB);
  }
</script>

<main>
  <h1><code>nft-trust-score</code></h1>

  <p>How trustworthy is an NFT relative to another?</p>

  {#if !trustScorePromise}
    <form on:submit|preventDefault={handleOnSubmit}>
      <div class="card">
        <input
          class="address-input"
          type="text"
          autocomplete="off"
          maxlength="42"
          bind:value={collectionAddressA}
          placeholder="Contract Address"
        />
      </div>
      <div class="card">
        <input
          class="address-input"
          type="text"
          autocomplete="off"
          maxlength="42"
          bind:value={collectionAddressB}
          placeholder="Contract Address"
        />
      </div>
      <div class="card">
        <input type="submit" value="Go" />
      </div>
      {#if formError !== ""}
        <div class="card">
          <p class="error-text">{formError}</p>
        </div>
      {/if}
    </form>
  {:else}
    {#await trustScorePromise}
      <div class="card">
        <p>Computing trust score...</p>
      </div>
    {:then trustScore}
      <div class="card">
        <h2>
          Trust Score:
          {trustScore.mutualOwnerCount}/{trustScore.mutualOriginatorCount}/{trustScore.mutualOwnerOriginatorCount}
        </h2>
        <p>
          These contracts have {trustScore.mutualOwnerCount} mutual token owners.
        </p>
        <p>
          These contracts have {trustScore.mutualOriginatorCount} mutual token minters.
        </p>
        <p>
          These contracts have {trustScore.mutualOwnerOriginatorCount} mutual token
          owners who are also token minters.
        </p>
      </div>
    {:catch error}
      <div class="card">
        <p class="error-text">{error.message}</p>
      </div>
    {/await}
  {/if}

  <p>
    Check it out on
    <a href="https://www.npmjs.com/package/nft-trust-score" target="_blank"
      >npm</a
    >
    and
    <a href="https://github.com/fjij/nft-trust-score#readme" target="_blank"
      >GitHub</a
    >!
  </p>

  <p class="footer">
    Made with ❤️ by <a href="https://github.com/fjij" target="_blank">fjij</a>
  </p>
</main>

<style>
  .address-input {
    font-family: monospace;
    width: 42ch;
  }
  .footer {
    color: #888;
  }
  .error-text {
    color: #f00;
  }
</style>
