# Solution Recipes

Step-by-step recipes that combine Liferay Community APAC Marketplace Apps, tutorials and Liferay DXP features to build a
complete use case, published at <https://liferay-community-apac.github.io/solution-recipes/>.

The recipes are Markdown pages in `docs/`, built with Jekyll and published by the shared Liferay Community APAC
workflow whenever `docs/` changes on `main`. The layout comes from the
[app docs template](https://github.com/Liferay-Community-APAC/app-docs-template), so the site shares the main
site's header, footer, styles and search.

## Add a recipe

1. Copy `RECIPE_TEMPLATE.md` to `docs/<recipe-name>/index.md`, for example `docs/community-qa-portal/index.md`.
2. Fill in the front matter:

    ```yaml
    ---
    title: Community Q&A Portal      # the recipe's name
    nav_order: 10                    # its position in the list, lower first
    recipe: true                     # lists it on the home page
    summary: One sentence for its card on the home page.
    description: One sentence for search results.
    ---
    ```

3. Write the four sections: **What You'll Build**, **Ingredients**, **Steps** and **Result**. Link each ingredient
   to its own guide instead of repeating it. Put screenshots next to the page, in `docs/<recipe-name>/`.
4. Open a pull request. The home page lists the recipe automatically.

If a recipe needs real code, such as a Liferay Workspace or client extensions, put the code in its own repository
and list it as an ingredient.

The `docs/example-recipe/` page is a placeholder. Delete it when the first real recipe is added.

## Preview locally

Set `main_site: https://liferay-community-apac.github.io` in `docs/_config.yml` (don't commit it), then:

```bash
cd docs && jekyll serve --baseurl /solution-recipes
```

## Publishing

In **Settings → Pages**, **Source** must be **GitHub Actions**.
