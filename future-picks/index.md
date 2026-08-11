---
layout: default
title: 2027 Draft Board
permalink: /future-picks/
---

{% assign teams = site.data.teams | sort: "short_name" %}
{% assign rounds = site.draft_rounds | default: 18 %}
{% assign pick_trades = site.data.pick_trades %}

<div class="fp-header">
  <div>
    <p class="fp-eyebrow">Future Pick Ledger</p>
    <h1>2027 Draft Board</h1>
    <p class="fp-sub">Columns = original team. Cell = who picks there now.</p>
  </div>
  <div class="fp-badges">
    <span class="badge badge-strong">{{ site.draft_year }} Draft</span>
    <span class="badge">{{ rounds }} Rounds</span>
  </div>
</div>

{% if site.trade_house_rule %}
<div class="fp-rule">
  <strong>House rule:</strong> {{ site.trade_house_rule }}
</div>
{% endif %}

<div class="fp-board-wrap">
  <table class="fp-board">
    <thead>
      <tr>
        <th class="fp-rd-col">Rd</th>
        {% for team in teams %}
          <th>{{ team.short_name | default: team.owner }}</th>
        {% endfor %}
      </tr>
    </thead>
    <tbody>
      {% for r in (1..rounds) %}
      <tr>
        <td class="fp-rd-col">{{ r }}</td>
        {% for col_team in teams %}
          {% assign matches = pick_trades | where: "round", r | where: "original_team", col_team.id %}
          {% if matches.size > 0 %}
            {% assign current_trade = matches | last %}
            {% assign current_team = teams | where: "id", current_trade.new_team | first %}
            <td class="fp-cell fp-traded" title="{% if current_trade.note %}{{ current_trade.note }} — {% endif %}{{ current_trade.date }}">
              <span class="fp-orig">{{ col_team.short_name | default: col_team.owner }}</span>
              <span class="fp-new">{{ current_team.short_name | default: current_team.owner }}</span>
            </td>
          {% else %}
            <td class="fp-cell">{{ col_team.short_name | default: col_team.owner }}</td>
          {% endif %}
        {% endfor %}
      </tr>
      {% endfor %}
    </tbody>
  </table>
</div>

<div class="fp-lower">
  <section class="fp-panel">
    <h2>File a Trade</h2>
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
    <p class="fp-sub">Copy this block into <code>_data/pick_trades.yml</code> (GitHub → edit file → paste above the closing <code>]</code> or replace it if empty), then commit to <code>main</code>.</p>
  </section>

  <section class="fp-panel">
    <h2>Trade Log</h2>
    {% if pick_trades.size == 0 %}
      <p class="fp-sub">No trades filed yet. This board is clean — every team holds its own picks.</p>
    {% else %}
      {% assign sorted_trades = pick_trades | sort: "date" | reverse %}
      <ul class="fp-log">
        {% for t in sorted_trades %}
          {% assign orig = teams | where: "id", t.original_team | first %}
          {% assign dest = teams | where: "id", t.new_team | first %}
          <li>
            <span class="fp-log-date">{{ t.date | date: "%b %-d, %Y" }}</span>
            Round {{ t.round }}: <strong>{{ orig.short_name | default: orig.owner }}</strong>'s pick →
            <strong>{{ dest.short_name | default: dest.owner }}</strong>
            {% if t.note %}<span class="fp-log-note">{{ t.note }}</span>{% endif %}
          </li>
        {% endfor %}
      </ul>
    {% endif %}
  </section>
</div>

<section class="fp-panel fp-counts">
  <h2>Pick Counts</h2>
  <p class="fp-sub">Should equal {{ rounds }} per team. Off-count teams need a roster plan for the extra/missing slot.</p>
  <div class="fp-count-list">
    {% for team in teams %}
      {% assign count = 0 %}
      {% for r in (1..rounds) %}
        {% for col_team in teams %}
          {% assign current_id = col_team.id %}
          {% assign matches = pick_trades | where: "round", r | where: "original_team", col_team.id %}
          {% if matches.size > 0 %}
            {% assign current_id = matches.last.new_team %}
          {% endif %}
          {% if current_id == team.id %}
            {% assign count = count | plus: 1 %}
          {% endif %}
        {% endfor %}
      {% endfor %}
      <div class="fp-count-row">
        <span>{{ team.short_name | default: team.owner }}</span>
        <span class="fp-count-badge {% if count != rounds %}fp-count-off{% endif %}">{{ count }}</span>
      </div>
    {% endfor %}
  </div>
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
