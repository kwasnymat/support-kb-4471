<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Problem = 'parser' | 'hint' | 'one-day move';

  let { ticket, issues }: { ticket: number; issues: Record<Problem, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rows = $derived(
    [
      ...groupByCause(merged, tenantOf).map((g) => ({
        problem: g.cause as Problem,
        reports: g.reports,
      })),
      { problem: 'one-day move' as Problem, reports: moved },
    ].sort((a, b) => b.reports.length - a.reports.length),
  );
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.reports.length}</td>
        <td>{new Set(row.reports.map((r) => r.tenant)).size}</td>
        <td>{seen(row.reports)}</td>
        <td><a href={issues[row.problem]}>{issues[row.problem].replace(/^.*\/issues\//, '#')}</a></td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  The {moved.length} <code>one-day move</code> reports are also counted under <code>parser</code>, where the desk files
  them by tenant.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
