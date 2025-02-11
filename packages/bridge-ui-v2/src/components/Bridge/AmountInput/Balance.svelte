<script lang="ts">
  import type { FetchBalanceResult } from '@wagmi/core';
  import { t } from 'svelte-i18n';

  import LoadingText from '$components/LoadingText/LoadingText.svelte';
  import { getBalance as getTokenBalance, type Token } from '$libs/token';
  import { truncateString } from '$libs/util/truncateString';
  import { type Account, account } from '$stores/account';
  import { network } from '$stores/network';

  import { destNetwork, selectedToken } from '../state';

  let tokenBalance: Maybe<FetchBalanceResult>;
  let computingTokenBalance = false;
  let errorComputingTokenBalance = false;

  async function updateTokenBalance(token: Maybe<Token>, account?: Account, srcChainId?: number, destChainId?: number) {
    if (!token || !account || !account.address) return;

    computingTokenBalance = true;
    errorComputingTokenBalance = false;

    try {
      tokenBalance = await getTokenBalance({
        token,
        destChainId,
        userAddress: account.address,
        chainId: srcChainId,
      });
    } catch (error) {
      console.error(error);
      errorComputingTokenBalance = true;
    } finally {
      computingTokenBalance = false;
    }
  }

  export function renderTokenBalance(balance: Maybe<FetchBalanceResult>) {
    if (!balance) return '0.00';
    return `${truncateString(balance.formatted, 6)} ${balance.symbol}`;
  }

  $: updateTokenBalance($selectedToken, $account, $network?.id, $destNetwork?.id);
</script>

<div class="body-small-regular">
  <span>{$t('amount_input.balance')}:</span>
  <span>
    {#if computingTokenBalance}
      <LoadingText mask="0.0000" />
      <LoadingText mask="XXX" />
    {:else}
      {renderTokenBalance(tokenBalance)}
    {/if}
  </span>
</div>
