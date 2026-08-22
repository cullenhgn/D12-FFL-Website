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
    <p class="fp-sub">Generates the entries to paste into <code>_data/pick_trades.yml</code> — this page isn't linked anywhere in the site nav.</p>
  </div>
</div>

<section class="fp-panel">
  <p class="fp-sub">Pick the two teams in the trade, then list every pick moving each direction. A trade where Team A sends two picks and Team B sends one still just needs each pick listed once — you don't need to balance the counts.</p>

  <div class="ft-teams-row">
    <label>Team A
      <select id="ft-team-a"></select>
    </label>
    <label>Team B
      <select id="ft-team-b"></select>
    </label>
  </div>

  <div class="ft-side">
    <h3 id="ft-a-to-b-label">Picks Team A sends to Team B</h3>
    <div id="ft-a-to-b-rows"></div>
    <button type="button" class="ft-add-btn" id="ft-add-a-to-b">+ Add a pick</button>
  </div>

  <div class="ft-side">
    <h3 id="ft-b-to-a-label">Picks Team B sends to Team A</h3>
    <div id="ft-b-to-a-rows"></div>
    <button type="button" class="ft-add-btn" id="ft-add-b-to-a">+ Add a pick</button>
  </div>

  <label>Date
    <input type="date" id="ft-date">
  </label>
  <label>Note (optional, applied to every pick in this trade)
    <input type="text" id="ft-note" placeholder="e.g. part of a package for a WR">
  </label>

  <button id="ft-generate" type="button">Generate entries</button>

  <textarea id="ft-output" readonly rows="10" placeholder="Your YAML entries will appear here..."></textarea>
  <p class="fp-sub">Copy this whole block into <code>_data/pick_trades.yml</code>, pasted as new top-level entries (each starting with <code>- round:</code> at the very left edge, same indentation as the ones already there). Then commit to <code>main</code>. Once merged, the <a href="{{ '/future-picks/' | relative_url }}">Draft Board</a> and Trade Log update automatically.</p>
</section>

<script>
(function () {
  var teams = {{ teams | jsonify }};
  var roundCount = {{ rounds }};

  var teamASel = document.getElementById('ft-team-a');
  var teamBSel = document.getElementById('ft-team-b');
  var aToBRows = document.getElementById('ft-a-to-b-rows');
  var bToARows = document.getElementById('ft-b-to-a-rows');
  var aToBLabel = document.getElementById('ft-a-to-b-label');
  var bToALabel = document.getElementById('ft-b-to-a-label');
  var dateInput = document.getElementById('ft-date');
  var noteInput = document.getElementById('ft-note');
  var output = document.getElementById('ft-output');

  function teamLabel(id) {
    var t = teams.filter(function (x) { return x.id === id; })[0];
    return t ? (t.short_name || t.owner || id) : id;
  }

  function fillTeamSelect(sel) {
    teams.forEach(function (t) {
      var opt = document.createElement('option');
      opt.value = t.id;
      opt.textContent = t.short_name || t.owner || t.id;
      sel.appendChild(opt);
    });
  }
  fillTeamSelect(teamASel);
  fillTeamSelect(teamBSel);
  if (teams.length > 1) teamBSel.selectedIndex = 1;

  function addPickRow(container, defaultOriginalTeamId) {
    var row = document.createElement('div');
    row.className = 'ft-pick-row';

    var roundSel = document.createElement('select');
    roundSel.className = 'ft-round-select';
    for (var r = 1; r <= roundCount; r++) {
      var opt = document.createElement('option');
      opt.value = r;
      opt.textContent = 'Round ' + r;
      roundSel.appendChild(opt);
    }

    var origSel = document.createElement('select');
    origSel.className = 'ft-orig-select';
    teams.forEach(function (t) {
      var opt = document.createElement('option');
      opt.value = t.id;
      opt.textContent = 'Originally ' + (t.short_name || t.owner || t.id) + "'s pick";
      if (t.id === defaultOriginalTeamId) opt.selected = true;
      origSel.appendChild(opt);
    });

    var removeBtn = document.createElement('button');
    removeBtn.type = 'button';
    removeBtn.className = 'ft-remove-btn';
    removeBtn.textContent = '✕';
    removeBtn.addEventListener('click', function () {
      if (container.children.length > 1) row.remove();
    });

    row.appendChild(roundSel);
    row.appendChild(origSel);
    row.appendChild(removeBtn);
    container.appendChild(row);
  }

  function refreshLabelsAndDefaults() {
    var aId = teamASel.value;
    var bId = teamBSel.value;
    aToBLabel.textContent = 'Picks ' + teamLabel(aId) + ' sends to ' + teamLabel(bId);
    bToALabel.textContent = 'Picks ' + teamLabel(bId) + ' sends to ' + teamLabel(aId);
  }

  document.getElementById('ft-add-a-to-b').addEventListener('click', function () {
    addPickRow(aToBRows, teamASel.value);
  });
  document.getElementById('ft-add-b-to-a').addEventListener('click', function () {
    addPickRow(bToARows, teamBSel.value);
  });

  teamASel.addEventListener('change', refreshLabelsAndDefaults);
  teamBSel.addEventListener('change', refreshLabelsAndDefaults);

  // start with one row each direction
  addPickRow(aToBRows, teamASel.value);
  addPickRow(bToARows, teamBSel.value);
  refreshLabelsAndDefaults();

  document.getElementById('ft-generate').addEventListener('click', function () {
    var aId = teamASel.value;
    var bId = teamBSel.value;
    var date = dateInput.value || new Date().toISOString().slice(0, 10);
    var note = noteInput.value.trim();

    if (aId === bId) {
      output.value = 'Team A and Team B need to be two different teams.';
      return;
    }

    var entries = [];

    function collectRows(container, newTeamId) {
      Array.prototype.forEach.call(container.children, function (row) {
        var round = row.querySelector('.ft-round-select').value;
        var origTeam = row.querySelector('.ft-orig-select').value;
        entries.push({ round: round, original_team: origTeam, new_team: newTeamId });
      });
    }

    collectRows(aToBRows, bId);
    collectRows(bToARows, aId);

    if (entries.length === 0) {
      output.value = 'Add at least one pick to the trade.';
      return;
    }

    var lines = [];
    entries.forEach(function (e) {
      lines.push('- round: ' + e.round);
      lines.push('  original_team: ' + e.original_team);
      lines.push('  new_team: ' + e.new_team);
      lines.push('  date: ' + date);
      if (note) {
        lines.push('  note: "' + note.replace(/"/g, '\\"') + '"');
      }
    });
    output.value = lines.join('\n');
  });
})();
</script>
