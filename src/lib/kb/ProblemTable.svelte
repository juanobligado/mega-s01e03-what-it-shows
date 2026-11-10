<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { splitTicket, type Problem } from '#lib/kb/problems';

  // tickets: problem → link to the ticket filed for it.
  let { ticket, tickets }: { ticket: number; tickets: Partial<Record<Problem, { label: string; href: string }>> } = $props();

  const LABEL: Record<Problem, string> = {
    parser: 'shared parser reads day-first dates month first',
    hint: 'booking-v3 form hint says MM/DD, form reads DD/MM',
    'one-day': 'date moves by one day (time zone), not swapped',
    unplaced: 'no tenant on file',
  };

  const split = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const tags = (t: Record<string, number>) =>
    Object.entries(t)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{split.merged.length}</strong> unique reports merged into #{ticket} ·
  <strong>{split.groups.length}</strong> {split.groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>intake tagged</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code><br /><small>{LABEL[g.problem]}</small></td>
        <td><strong>{g.reports.length}</strong></td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td class="tags">{tags(g.taggedAs)}</td>
        <td>{#if tickets[g.problem]}<a href={tickets[g.problem]?.href}>{tickets[g.problem]?.label}</a>{:else}none{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .tags { color: var(--muted); font-size: 0.85em; }
  small { color: var(--muted); }
</style>
