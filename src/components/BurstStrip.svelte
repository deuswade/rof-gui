<script lang="ts">
  import type { AnalysisResult } from '../rof-detector.js';
  import {
    app, isBurstExcluded, toggleBurstExcluded, clearExcludedBursts, seekTo
  } from '../state.svelte.js';

  interface Props {
    results: AnalysisResult;
  }
  const { results }: Props = $props();

  const bursts = $derived(results.bursts || []);
  // Read app.excludedBursts so the flags recompute when exclusions change.
  const excluded = $derived(
    (void app.excludedBursts, bursts.map(b => isBurstExcluded(b)))
  );
  const excludedCount = $derived(excluded.filter(Boolean).length);

  function fmt(n: number, d = 1): string {
    return Number.isFinite(n) ? n.toFixed(d) : '—';
  }
</script>

{#if bursts.length > 0}
  <section class="strip">
    <div class="strip-header">
      <h3>Per burst</h3>
      <div class="strip-meta">
        {#if excludedCount > 0}
          <span class="count">{bursts.length - excludedCount} of {bursts.length} bursts included</span>
          <button class="restore-all" onclick={clearExcludedBursts}>Restore all</button>
        {:else}
          <span class="count">{bursts.length} burst{bursts.length !== 1 ? 's' : ''}</span>
        {/if}
      </div>
    </div>
    <div class="cards">
      {#each bursts as b, i}
        <div class="card" class:excluded={excluded[i]}>
          <div class="card-top">
            <button
              class="card-id"
              onclick={() => seekTo(b.startTime)}
              title="Jump to this burst on the timeline"
            >#{b.burstNumber}{#if excluded[i]}<span class="card-tag">excluded</span>{:else if b.added}<span class="card-tag added">added</span>{/if}</button>
            <button
              class="card-toggle"
              onclick={() => toggleBurstExcluded(b)}
              aria-label={excluded[i] ? `Restore burst ${b.burstNumber}` : `Exclude burst ${b.burstNumber}`}
              title={excluded[i] ? 'Count this burst again' : 'Leave this burst out of the results'}
            >{excluded[i] ? 'Restore' : 'Exclude'}</button>
          </div>
          <div class="card-rpm">
            {Math.round(b.rateRpm)}
            <span class="card-rpm-unit">RPM</span>
          </div>
          {#if b.rateRpmCI95 > 0}
            <div class="card-ci">±{fmt(b.rateRpmCI95)}</div>
          {/if}
          <div class="card-row">
            <span class="card-label">shots</span>
            <span class="card-val">{b.numShots}</span>
          </div>
          <div class="card-row">
            <span class="card-label">window</span>
            <span class="card-val">{fmt(b.startTime, 2)}–{fmt(b.endTime, 2)}s</span>
          </div>
          <div class="card-row">
            <span class="card-label">interval</span>
            <span class="card-val">{fmt(b.meanInterval * 1000)} ms</span>
          </div>
        </div>
      {/each}
    </div>
  </section>
{/if}

<style>
  .strip {
    background: var(--bg-elev);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 18px 20px 20px;
  }

  .strip-header {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 6px 12px;
    margin-bottom: 14px;
  }

  .strip-header h3 {
    margin: 0;
    font-size: 13px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: var(--text-secondary);
    white-space: nowrap;
  }

  .strip-meta {
    display: flex;
    align-items: baseline;
    gap: 10px;
  }

  .restore-all {
    background: transparent;
    border: 1px solid var(--border-strong);
    color: var(--text-secondary);
    padding: 3px 8px;
    border-radius: var(--radius);
    font-size: 11px;
    transition: all 0.12s;
  }

  .restore-all:hover {
    border-color: var(--accent);
    color: var(--accent);
  }

  .count {
    font-family: var(--font-mono);
    font-size: 12px;
    color: var(--text-tertiary);
  }

  .cards {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 10px;
  }

  .card {
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 12px 14px;
    display: flex;
    flex-direction: column;
    gap: 6px;
    transition: border-color 0.12s;
  }

  .card:hover {
    border-color: var(--accent);
  }

  .card.excluded {
    border-style: dashed;
    background: repeating-linear-gradient(
      -45deg,
      transparent 0 6px,
      rgba(31, 41, 55, 0.035) 6px 12px
    );
  }

  .card.excluded:hover {
    border-color: var(--border-strong);
  }

  .card.excluded .card-rpm,
  .card.excluded .card-ci,
  .card.excluded .card-row {
    opacity: 0.4;
  }

  .card.excluded .card-rpm {
    text-decoration: line-through;
    text-decoration-thickness: 2px;
  }

  /* Inline-block stops the strikethrough from running through the unit. */
  .card.excluded .card-rpm-unit {
    display: inline-block;
    text-decoration: none;
  }

  .card-top {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
  }

  .card-id {
    background: none;
    border: none;
    padding: 0;
    font-family: var(--font-mono);
    font-size: 11px;
    color: var(--text-tertiary);
    letter-spacing: 0.04em;
  }

  .card-id:hover {
    color: var(--accent);
  }

  .card-tag {
    margin-left: 6px;
    text-transform: uppercase;
    font-size: 10px;
    color: var(--danger);
  }

  .card-tag.added {
    color: var(--accent);
  }

  .card-toggle {
    background: transparent;
    border: 1px solid var(--border-strong);
    color: var(--text-secondary);
    padding: 2px 8px;
    border-radius: var(--radius);
    font-size: 11px;
    transition: all 0.12s;
  }

  .card-toggle:hover {
    border-color: var(--danger);
    color: var(--danger);
  }

  .card.excluded .card-toggle {
    border-color: var(--accent);
    color: var(--accent);
  }

  .card.excluded .card-toggle:hover {
    background: var(--accent);
    color: white;
  }

  .card-rpm {
    font-family: var(--font-mono);
    font-size: 28px;
    font-weight: 700;
    color: var(--text);
    line-height: 1;
    font-variant-numeric: tabular-nums;
    margin-top: 2px;
  }

  .card-rpm-unit {
    font-size: 12px;
    color: var(--text-tertiary);
    font-weight: 500;
    margin-left: 4px;
  }

  .card-ci {
    font-family: var(--font-mono);
    font-size: 11px;
    color: var(--text-tertiary);
    margin-top: -2px;
  }

  .card-row {
    display: flex;
    justify-content: space-between;
    font-size: 12px;
    margin-top: 2px;
  }

  .card-label {
    color: var(--text-tertiary);
    text-transform: uppercase;
    letter-spacing: 0.04em;
    font-size: 10px;
  }

  .card-val {
    font-family: var(--font-mono);
    color: var(--text-secondary);
    font-variant-numeric: tabular-nums;
  }
</style>
