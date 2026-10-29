<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in a ticket, each with the issue that tracks it. Counted the desk's way:
  // a report sits under the cause its tenant gives it; reports the desk read as a one-day move stay
  // counted there and get a row of their own as well (desk-notes.json).
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const rows = $derived([
    ...groups.map((g) => ({ problem: g.cause as string, reports: g.reports.length, within: null as string | null })),
    ...groups
      .filter((g) => readAs(g, notes.oneDayMove))
      .map((g) => ({ problem: 'one-day move', reports: readAs(g, notes.oneDayMove), within: g.cause as string })),
  ]);
  const issueNo = (url: string) => `#${url.split('/').pop()}`;
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong>
  {rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>issue</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr class:open={!issues[row.problem]}>
        <td><code>{row.problem}</code>{#if row.within}<span class="within"> · counted in <code>{row.within}</code></span>{/if}</td>
        <td>{row.reports}</td>
        <td>{#if issues[row.problem]}<a href={issues[row.problem]}>{issueNo(issues[row.problem])}</a>{:else}no issue yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .within { color: var(--muted); font-size: 0.85em; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
