---
title: Solution Recipes
nav_order: 1
description: Step-by-step recipes that combine Liferay Community APAC apps, tutorials and Liferay DXP features to build a complete use case.
---

# Solution Recipes
{: .no_toc }

Each recipe combines apps, tutorials and Liferay DXP features into one working use case: what you'll build, the ingredients you need, and the steps to put them together.

{: .important }
Recipes are unofficial guides shared by the Liferay Customer Success APAC team, like the [apps and tutorials](https://liferay-community-apac.github.io/) they use. Test them in a non-production environment before relying on them.

## Recipes

{%- assign recipes = site.pages | where: "recipe", true | sort: "nav_order", "last" %}
<div class="doc-cards">
{%- for r in recipes %}
  <a class="card doc" href="{{ r.url | relative_url }}"><strong>{{ r.title }}{% if r.status == "placeholder" %} (placeholder){% endif %}</strong><span>{{ r.summary }}</span></a>
{%- endfor %}
</div>

## How a Recipe Is Organized

Every recipe follows the same structure:

1. **What you'll build**: the use case and the end result.
1. **Ingredients**: the apps, tutorials and Liferay DXP features it uses, each linking to its guide.
1. **Steps**: how to combine them, in order.
1. **Result**: how to check that it works, and where to go next.

To suggest or contribute a recipe, open an issue or a pull request in the [solution-recipes repository](https://github.com/Liferay-Community-APAC/solution-recipes).
