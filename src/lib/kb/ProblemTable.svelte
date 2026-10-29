<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in a ticket, each with the issue that tracks it. A report the desk read as a
  // one-day move is not a flipped date, whatever its tenant: it gets its own row, not a footnote.
  type Problem = 'parser' | 'hint' | 'one-day move' | 'unplaced';
  let { ticket, issues }: { ticket: number; issues: Partial<Record<Problem, string>> } = $props();

  const problemOf = (r: Report): Problem =>
    notes.oneDayMove.includes(r.id) ? 'one-day move' : (causeOf(r, tenantOf(r)) ?? 'unplaced');

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    Object.entries(Object.groupBy(merged, problemOf))
      .map(([problem, rs = []]) => {
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        return {
          problem: problem as Problem,
          reports: rs.length,
          tenants: new Set(rs.map((r) => r.tenant)).size,
          seen: `${days[0]} → ${days[days.length - 1]}`,
        };
      })
      .sort((a, b) => b.reports - a.reports),
  );
  const issueNo = (url: string) => `#${url.split('/').pop()}`;
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong>
  {rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>issue</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr class:open={!issues[row.problem]}>
        <td><code>{row.problem}</code></td>
        <td>{row.reports}</td>
        <td>{row.tenants}</td>
        <td>{row.seen}</td>
        <td>{#if issues[row.problem]}<a href={issues[row.problem]}>{issueNo(issues[row.problem]!)}</a>{:else}no issue yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
