---
title: Solution Recipes
nav_order: 1
description: Step-by-step recipes that combine Liferay Community APAC Marketplace Apps, tutorials and Liferay DXP features to build a complete use case.
---

# Solution Recipes
{: .no_toc }

Each recipe combines Marketplace Apps, tutorials and Liferay DXP features into one working use case: what you'll build, the ingredients you need, and the steps to put them together.

{: .important }
Recipes are unofficial guides, like the [Marketplace Apps and tutorials](https://liferay-community-apac.github.io/) they use. Test them in a non-production environment before relying on them.

## Recipes

{%- assign recipes = site.pages | where: "recipe", true | sort: "nav_order", "last" %}
{%- assign alltags = "" %}
{%- for r in recipes %}{% for t in r.tags %}{% assign alltags = alltags | append: "|" | append: t %}{% endfor %}{% endfor %}
{%- assign alltags = alltags | remove_first: "|" | split: "|" | uniq | sort_natural %}
{%- if alltags.size > 0 %}
<div class="recipe-filter" role="group" aria-label="Filter recipes by tag" hidden>
  <button type="button" data-tag="" aria-pressed="true">All <span class="n">{{ recipes.size }}</span></button>
  {%- for t in alltags %}
  {%- assign n = 0 %}{% for r in recipes %}{% if r.tags contains t %}{% assign n = n | plus: 1 %}{% endif %}{% endfor %}
  <button type="button" data-tag="{{ t | escape }}" aria-pressed="false">{{ t }} <span class="n">{{ n }}</span></button>
  {%- endfor %}
</div>
<p class="recipe-count" aria-live="polite"></p>
{%- endif %}
<div class="doc-cards">
{%- for r in recipes %}
  <a class="card doc" href="{{ r.url | relative_url }}" data-tags="|{{ r.tags | join: '|' | escape }}|">{% if r.product or r.apps %}<span class="card-logos">{% if r.product %}<img class="card-logo" src="{{ r.product.logo | relative_url }}" alt="" width="56" height="56">{% endif %}{% for app in r.apps %}<img class="card-app-logo" src="{{ app.logo | relative_url }}" alt="" width="28" height="28">{% endfor %}</span>{% endif %}<strong>{{ r.title }}{% if r.status == "placeholder" %} (placeholder){% endif %}</strong><span>{{ r.summary }}</span>{% if r.tags %}<span class="recipe-tags">{% for t in r.tags %}<span class="rtag">{{ t }}</span>{% endfor %}</span>{% endif %}</a>
{%- endfor %}
</div>

<script>
  // Recipe filter: show the recipes with the chosen tag. The tag is kept in the address (?tag=…), so a
  // tag on a recipe page links straight to the filtered list.
  (function () {
    var bar = document.querySelector('.recipe-filter');
    if (!bar) return;
    var cards = [].slice.call(document.querySelectorAll('.doc-cards .card.doc[data-tags]'));
    var count = document.querySelector('.recipe-count');
    function apply(tag, push) {
      var shown = 0;
      cards.forEach(function (c) {
        var on = !tag || c.getAttribute('data-tags').indexOf('|' + tag + '|') !== -1;
        c.hidden = !on; if (on) shown++;
      });
      [].forEach.call(bar.querySelectorAll('button'), function (b) { b.setAttribute('aria-pressed', b.getAttribute('data-tag') === tag ? 'true' : 'false'); });
      count.textContent = tag ? shown + (shown === 1 ? ' recipe' : ' recipes') + ' tagged \u201c' + tag + '\u201d' : '';
      if (push) {
        var u = new URL(location.href);
        if (tag) u.searchParams.set('tag', tag); else u.searchParams.delete('tag');
        history.replaceState(null, '', u);
      }
    }
    bar.addEventListener('click', function (e) {
      var b = e.target.closest('button'); if (b) apply(b.getAttribute('data-tag'), true);
    });
    bar.hidden = false;
    var start = new URL(location.href).searchParams.get('tag') || '';
    if (start && !bar.querySelector('button[data-tag="' + start.replace(/"/g, '\\"') + '"]')) start = '';
    apply(start, false);
  })();
</script>

{%- assign shown = "" %}
{%- for r in recipes %}
{%- if r.product and shown contains r.product.name %}{% continue %}{% endif %}
{%- if r.product %}
{%- assign shown = shown | append: "|" | append: r.product.name %}
{% include product-spotlight.html product=r.product %}
{%- endif %}
{%- endfor %}

## How a Recipe Is Organized

Every recipe follows the same structure:

1. **What you'll build**: the use case and the end result.
1. **Ingredients**: the Marketplace Apps, tutorials and Liferay DXP features it uses, each linking to its guide.
1. **Steps**: how to combine them, in order.
1. **Result**: how to check that it works, and where to go next.

To suggest or contribute a recipe, open an issue or a pull request in the [solution-recipes repository](https://github.com/Liferay-Community-APAC/solution-recipes).
