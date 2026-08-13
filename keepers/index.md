---
layout: default
title: Keeper History
permalink: /keepers/
---

<div class="fp-header">
  <div>
    <h1>Keeper History</h1>
    <p class="fp-sub">Every keeper, every year, and the draft round it cost.</p>
  </div>
</div>

<div id="kp-tabs" class="pd-tabs"></div>

<div class="pd-legend">
  <span class="pd-legend-item pd-pos-QB">QB</span>
  <span class="pd-legend-item pd-pos-RB">RB</span>
  <span class="pd-legend-item pd-pos-WR">WR</span>
  <span class="pd-legend-item pd-pos-TE">TE</span>
  <span class="pd-legend-item pd-keeper-legend">* = cost increased by keeping multiple players at the same round</span>
</div>

<section id="kp-list-wrap" class="fp-panel">
  <p class="fp-sub">No keeper data logged yet. Add a year to <code>_data/keepers.yml</code> to see it here.</p>
</section>

<script>
(function () {
  var years = {{ site.data.keepers | jsonify }};

  if (!years || years.length === 0) return;

  years.sort(function (a, b) { return b.year - a.year; });

  var tabsEl = document.getElementById('kp-tabs');
  var listWrap = document.getElementById('kp-list-wrap');
  var defaultYear = 2025;
  var defaultIndex = 0;
  years.forEach(function (y, i) {
    if (y.year === defaultYear) defaultIndex = i;
  });

  years.forEach(function (yearData, i) {
    var btn = document.createElement('button');
    btn.textContent = yearData.year;
    btn.className = 'pd-tab' + (i === defaultIndex ? ' pd-tab-active' : '');
    btn.addEventListener('click', function () {
      document.querySelectorAll('.pd-tab').forEach(function (b) { b.classList.remove('pd-tab-active'); });
      btn.classList.add('pd-tab-active');
      renderList(yearData);
    });
    tabsEl.appendChild(btn);
  });

  function escapeHtml(str) {
    return String(str).replace(/[&<>"']/g, function (c) {
      return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c];
    });
  }

  function renderList(yearData) {
    var keepers = (yearData.keepers || []).slice().sort(function (a, b) { return a.round - b.round; });

    if (keepers.length === 0) {
      listWrap.innerHTML = '<p class="fp-sub">No keepers logged for ' + yearData.year + '.</p>';
      return;
    }

    var html = '<h2>' + yearData.year + ' Keepers</h2><ul class="fp-log">';
    keepers.forEach(function (k) {
      var posClass = 'kp-pos-badge pd-pos-' + (k.position || '').toUpperCase();
      html += '<li>';
      html += '<span class="fp-log-date">Round ' + k.round + (k.extra_cost ? ' *' : '') + '</span>';
      html += '<strong>' + escapeHtml(k.player) + '</strong> ';
      html += '<span class="' + posClass + '">' + escapeHtml(k.position || '') + '</span>';
      html += '</li>';
    });
    html += '</ul>';
    listWrap.innerHTML = html;
  }

  renderList(years[defaultIndex]);
})();
</script>
