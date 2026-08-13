---
layout: default
title: File a Trade
permalink: /file-a-trade/
---

{% assign teams = site.data.teams | sort: "draft_order" %}
{% assign rounds = site.draft_rounds | default: 18 %}

<div class="fp-header">
  <div>
    <h1>File a Trade</h1>
    <p class="fp-sub">Generates the entry to paste into <code>_data/pick_trades.yml</code> — this page isn't linked anywhere in the site nav.</p>
  </div>
</div>

<section class="fp-panel">
  <p class="fp-sub">This site is static, so trades aren't saved automatically — fill this out and it'll generate the entry to paste into <code>_data/pick_trades.yml</code>.</p>

  <label>Round
    <select id="fp-round"></select>
  </label>
  <label>Pick's original team (the column)
    <select id="fp-original"></select>
  </label>
  <label>New team (who's picking up the pick)
    <select id="fp-new"></select>
  </label>
  <label>Date
    <input type="date" id="fp-date">
  </label>
  <label>Note (optional)
    <input type="text" id="fp-note" placeholder="e.g. part of a package for a WR">
  </label>

  <button id="fp-generate" type="button">Generate entry</button>

  <textarea id="fp-output" readonly rows="6" placeholder="Your YAML entry will appear here..."></textarea>
  <p class="fp-sub">Copy this block into <code>_data/pick_trades.yml</code> (GitHub → edit file → paste above the closing <code>]</code> or replace it if empty), then commit to <code>main</code>. Once merged, the <a href="{{ '/future-picks/' | relative_url }}">Draft Board</a> and Trade Log update automatically.</p>
</section>

<script>
(function () {
  var teams = {{ teams | jsonify }};
  var roundCount = {{ rounds }};

  var roundSel = document.getElementById('fp-round');
  var origSel = document.getElementById('fp-original');
  var newSel = document.getElementById('fp-new');
  var dateInput = document.getElementById('fp-date');
  var noteInput = document.getElementById('fp-note');
  var output = document.getElementById('fp-output');

  for (var r = 1; r <= roundCount; r++) {
    var opt = document.createElement('option');
    opt.value = r;
    opt.textContent = 'Round ' + r;
    roundSel.appendChild(opt);
  }

  teams.forEach(function (t) {
    var label = t.short_name || t.owner || t.id;
    var o1 = document.createElement('option');
    o1.value = t.id;
    o1.textContent = label;
    origSel.appendChild(o1);

    var o2 = document.createElement('option');
    o2.value = t.id;
    o2.textContent = label;
    newSel.appendChild(o2);
  });

  document.getElementById('fp-generate').addEventListener('click', function () {
    var round = roundSel.value;
    var orig = origSel.value;
    var dest = newSel.value;
    var date = dateInput.value || new Date().toISOString().slice(0, 10);
    var note = noteInput.value.trim();

    if (orig === dest) {
      output.value = "Pick the original team and the new team as two different teams.";
      return;
    }

    var lines = [
      '- round: ' + round,
      '  original_team: ' + orig,
      '  new_team: ' + dest,
      '  date: ' + date
    ];
    if (note) {
      lines.push('  note: "' + note.replace(/"/g, '\\"') + '"');
    }
    output.value = lines.join('\n');
  });
})();
</script>
