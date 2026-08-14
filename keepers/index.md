---
layout: default
title: Keeper Logs
permalink: /keepers/
---

<div class="fp-header">
  <div>
    <h1>Keeper Logs</h1>
  </div>
</div>

<div class="pd-tabs">
  <button class="pd-tab pd-tab-active" type="button">2025</button>
</div>

<div class="pd-legend">
  <span class="pd-legend-item pd-keeper-legend">* = cost increased by keeping multiple players at the same round</span>
  <span class="pd-legend-item kp-triple-legend">Kept all 3 years</span>
</div>

<div id="kp-tables-wrap">
  <p class="fp-sub">No keeper data logged yet. Add players to <code>_data/keepers.yml</code> to see it here.</p>
</div>

<script>
(function () {
  var players = {{ site.data.keepers | jsonify }};
  var wrap = document.getElementById('kp-tables-wrap');
  var displayYears = ['2025', '2024', '2023'];
  var positionOrder = ['QB', 'RB', 'WR', 'TE'];
  var positionNames = { QB: 'Quarterbacks', RB: 'Running Backs', WR: 'Wide Receivers', TE: 'Tight Ends' };

  if (!players || players.length === 0) return;

  function escapeHtml(str) {
    return String(str).replace(/[&<>"']/g, function (c) {
      return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
    });
  }

  var html = '';

  positionOrder.forEach(function (pos) {
    var group = players.filter(function (p) { return (p.position || '').toUpperCase() === pos; });
    if (group.length === 0) return;

    group.sort(function (a, b) { return a.player.localeCompare(b.player); });

    html += '<section class="fp-panel kp-pos-table">';
    html += '<h2><span class="kp-pos-dot pd-pos-' + pos + '"></span>' + (positionNames[pos] || pos) + '</h2>';
    html += '<table class="kp-table"><thead><tr><th class="kp-name-col">Player</th>';
    displayYears.forEach(function (y) { html += '<th>' + y + '</th>'; });
    html += '</tr></thead><tbody>';

    group.forEach(function (p) {
      var keptAllThree = displayYears.every(function (y) {
        return p.rounds && p.rounds[y] && typeof p.rounds[y].round === 'number';
      });

      var nameClass = keptAllThree ? 'kp-name-col kp-triple-name' : 'kp-name-col';
      html += '<tr><td class="' + nameClass + '">' + escapeHtml(p.player) + '</td>';
      displayYears.forEach(function (y) {
        var entry = p.rounds ? p.rounds[y] : null;
        var cellClass = keptAllThree ? 'kp-cell kp-triple' : 'kp-cell';
        if (entry && typeof entry.round === 'number') {
          html += '<td class="' + cellClass + '">' + entry.round + (entry.extra_cost ? ' *' : '') + '</td>';
        } else if (entry && entry.ineligible) {
          html += '<td class="' + cellClass + '">—</td>';
        } else {
          html += '<td class="' + cellClass + '"></td>';
        }
      });
      html += '</tr>';
    });

    html += '</tbody></table></section>';
  });

  wrap.innerHTML = html;
})();
</script>
