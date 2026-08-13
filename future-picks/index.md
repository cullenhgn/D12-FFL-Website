---
layout: default
title: 2026 Draft Board
permalink: /future-picks/
---

{% assign teams = site.data.teams | sort: "draft_order" %}
{% assign rounds = site.draft_rounds | default: 18 %}
{% assign pick_trades = site.data.pick_trades %}

<div class="fp-header">
  <div>
    <h1>2026 Draft Board</h1>
    <p class="fp-sub">Columns = original team. Cell = who picks there now.</p>
  </div>
</div>

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

