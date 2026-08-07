---
layout: default
title: Draft Pick Trades
permalink: /trades/
---

# Draft Pick Trade Tracker

Every future-pick and keeper trade in league history, since ESPN doesn't
track these. Edit `_data/trades.yml` to add a new trade.

{% assign trades = site.data.trades | reverse %}
{% if trades.size == 0 %}
<p>No trades logged yet.</p>
{% endif %}

<div class="trade-list">
{% for trade in trades %}
  <div class="trade-card trade-{{ trade.status }}">
    <div class="trade-header">
      <span class="trade-date">{{ trade.date | date: "%B %-d, %Y" }}</span>
      <span class="trade-status">{{ trade.status | capitalize }}</span>
    </div>
    <div class="trade-teams">
      {% for team_id in trade.teams %}
        {% assign team = site.data.teams | where: "id", team_id | first %}
        <div class="trade-side">
          <h3>{{ team.team_name | default: team_id }}</h3>
          <ul>
            {% assign sends = trade.sends[team_id] %}
            {% for item in sends %}
              <li>{{ item }}</li>
            {% endfor %}
          </ul>
        </div>
      {% endfor %}
    </div>
    {% if trade.notes %}<p class="trade-notes">{{ trade.notes }}</p>{% endif %}
  </div>
{% endfor %}
</div>
