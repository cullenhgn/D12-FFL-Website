---
layout: default
title: Previous Drafts
permalink: /previous-drafts/
---

<div class="fp-header">
  <div>
    <p class="fp-eyebrow">Draft History</p>
    <h1>Previous Drafts</h1>
    <p class="fp-sub">Every pick, every year. Columns are draft slot order for that season.</p>
  </div>
</div>

<div id="pd-tabs" class="pd-tabs"></div>

<div class="pd-legend">
  <span class="pd-legend-item pd-pos-QB">QB</span>
  <span class="pd-legend-item pd-pos-RB">RB</span>
  <span class="pd-legend-item pd-pos-WR">WR</span>
  <span class="pd-legend-item pd-pos-TE">TE</span>
  <span class="pd-legend-item pd-pos-DEF">DEF</span>
  <span class="pd-legend-item pd-pos-K">K</span>
  <span class="pd-legend-item pd-keeper-legend"><span class="pd-keeper-badge">K</span> = Keeper</span>
</div>

<div id="pd-board-wrap" class="pd-board-wrap">
  <p class="fp-sub">No drafts logged yet. Add a year to <code>_data/drafts.yml</code> to see it here.</p>
</div>

<script>
(function () {
  var drafts = {{ site.data.drafts | jsonify }};
  var teams = {{ site.data.teams | jsonify }};

  var teamById = {};
  teams.forEach(function (t) { teamById[t.id] = t; });

  if (!drafts || drafts.length === 0) return;

  // Most recent year first
  drafts.sort(function (a, b) { return b.year - a.year; });

  var tabsEl = document.getElementById('pd-tabs');
  var boardWrap = document.getElementById('pd-board-wrap');

  drafts.forEach(function (draft, i) {
    var btn = document.createElement('button');
    btn.textContent = draft.year;
    btn.className = 'pd-tab' + (i === 0 ? ' pd-tab-active' : '');
    btn.addEventListener('click', function () {
      document.querySelectorAll('.pd-tab').forEach(function (b) { b.classList.remove('pd-tab-active'); });
      btn.classList.add('pd-tab-active');
      renderBoard(draft);
    });
    tabsEl.appendChild(btn);
  });

  function teamLabel(id) {
    var t = teamById[id];
    return t ? (t.short_name || t.owner || id) : id;
  }

  function renderBoard(draft) {
    var order = draft.team_order || [];
    var rounds = {};
    draft.picks.forEach(function (p) {
      rounds[p.round] = rounds[p.round] || {};
      rounds[p.round][p.team] = p;
    });
    var roundNums = Object.keys(rounds).map(Number).sort(function (a, b) { return a - b; });

    var html = '<div class="pd-board-scroll"><table class="pd-board">';
    html += '<thead><tr>';
    order.forEach(function (teamId) {
      html += '<th>' + escapeHtml(teamLabel(teamId)) + '</th>';
    });
    html += '</tr></thead><tbody>';

    roundNums.forEach(function (r) {
      html += '<tr>';
      order.forEach(function (teamId, colIndex) {
        var pick = rounds[r][teamId];
        var posClass = pick ? 'pd-pos-' + (pick.position || '').toUpperCase() : '';
        var odd = r % 2 === 1;
        var isTurn = odd ? colIndex === order.length - 1 : colIndex === 0;
        var arrow = isTurn ? '&darr;' : (odd ? '&rarr;' : '&larr;');

        html += '<td class="pd-cell ' + posClass + (pick && pick.keeper ? ' pd-keeper' : '') + '">';
        if (pick) {
          html += '<div class="pd-pick-num">' + r + '.' + (colIndex + 1) + (pick.keeper ? ' <span class="pd-keeper-badge">K</span>' : '') + '</div>';
          html += '<div class="pd-player">' + escapeHtml(pick.player) + '</div>';
          html += '<div class="pd-meta">' + escapeHtml(pick.position || '') + (pick.nfl_team ? ' - ' + escapeHtml(pick.nfl_team) : '') + '</div>';
          html += '<div class="pd-arrow">' + arrow + '</div>';
        } else {
          html += '<div class="pd-empty">&mdash;</div>';
        }
        html += '</td>';
      });
      html += '</tr>';
    });

    html += '</tbody></table></div>';
    boardWrap.innerHTML = html;
  }

  function escapeHtml(str) {
    return String(str).replace(/[&<>"']/g, function (c) {
      return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
    });
  }

  renderBoard(drafts[0]);
})();
</script>
