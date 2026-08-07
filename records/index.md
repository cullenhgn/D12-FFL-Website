---
layout: default
title: League Records
permalink: /records/
---

# Historical Stats & League Records

Edit `_data/records.yml` to add or update records.

{% for group in site.data.records %}
  <h2>{{ group.category }}</h2>
  <table class="records-table">
    <tbody>
    {% for r in group.records %}
      <tr>
        <td class="record-title">
          {% if r.title %}{{ r.title }}{% elsif r.detail %}{{ r.detail }}{% endif %}
          {% if r.season %}<span class="record-season">({{ r.season }}{% if r.week %}, Wk {{ r.week }}{% endif %})</span>{% endif %}
        </td>
        <td class="record-holder">{{ r.holder }}</td>
        {% if r.value %}<td class="record-value">{{ r.value }}</td>{% endif %}
      </tr>
    {% endfor %}
    </tbody>
  </table>
{% endfor %}
