# Agent Instructions

This repository is a Jekyll-based GitHub Pages site using the `minima` theme. 

## Roadmap & Modernization Decisions (April 2026)
- **Current Strategy:** Modernize within Jekyll (Option 2).
- **Future Direction:** Possible migration to Astro (Option 3) has been discussed but deferred.
- **Key Changes in Progress:** Moving to standard YAML front matter, migrating to GitHub Actions for deployment, and adopting build-time syntax highlighting (Rouge).

## High-Signal Quirks & Conventions

- **Front Matter Required:** All posts MUST have standard YAML front matter (`---`) with a `date` and `layout: post`. 
- **Titles from Headings:** The `jekyll-titles-from-headings` plugin is ENABLED. Do NOT put a `title` in the front matter. Instead, the very first line of the markdown body must be the title formatted as a Level 2 header (e.g., `## My Title`). This heading will be extracted as the page title and stripped from the post content.
- **Syntax Highlighting Transition:** Moving from client-side `highlight.js` to server-side Rouge. Check `_config.yml` for the current status of `kramdown.syntax_highlighter`.
- **Asset Paths / Images Transition:** Moving images from `_posts/` sidecar folders to `/assets/images/`. Use relative paths or `{{ site.baseurl }}` instead of absolute GitHub URLs.
- **Custom Styling:** Custom CSS overrides are located in `css/override.css`. Custom templates live in `_layouts/` and `_includes/`.

## Local Development

The project uses a `Gemfile`. Run `bundle install` followed by `bundle exec jekyll serve` to test locally.
