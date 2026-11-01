# Copilot instructions

## Repository shape and build flow

This repository is a GitHub Pages knowledge-base site, not a standalone Quartz source checkout. The root `content/` directory contains the published Markdown; the nested `quartz/` directory is the Quartz generator submodule. Root `quartz.config.ts` and `quartz.layout.ts` are this site's overrides.

The GitHub Actions workflow copies the root config, layout, and content into the submodule, builds the site there, and deploys `quartz/public` to GitHub Pages. Keep content and site-specific changes in the root paths; changes made only in the submodule's staged config/content can be overwritten by CI.

For a local preview, run these PowerShell commands from the repository root, then open the URL reported by Quartz (normally `http://localhost:8080/`):

```powershell
Copy-Item -Path .\quartz.config.ts, .\quartz.layout.ts -Destination .\quartz\ -Force
Copy-Item -Path .\content\* -Destination .\quartz\content\ -Recurse -Force
Set-Location .\quartz
npm ci
npx quartz build --serve
```

The generator requires Node.js 22 or newer and npm 10.9.2 or newer.

## Commands

Run Quartz commands from the `quartz/` directory:

| Purpose | Command |
| --- | --- |
| Build the staged site | `npx quartz build` |
| Preview with hot reload | `npx quartz build --serve` |
| Run all Quartz tests | `npm test` |
| Run one test by name | `npx tsx --test --test-name-pattern="slugifyFilePath" quartz/util/path.test.ts` |
| Type-check and check formatting | `npm run check` |
| Format Quartz source | `npm run format` |

`npm run check` runs `tsc --noEmit` followed by Prettier in check mode. Quartz tests use Node's test runner through `tsx`; test files are colocated with the code they cover and use `*.test.ts`.

## Content and site conventions

- Add or edit published pages under the root `content/`, using Markdown files and folders. The `content/` tree is the source copied into the generator during deployment.
- The site enables Obsidian-flavored Markdown, GitHub-flavored Markdown, and shortest-match internal link resolution. Preserve existing wikilink/frontmatter conventions when editing notes.
- Drafts are removed during generation. The root config also ignores `private`, `_templates`, and `.obsidian`; avoid relying on those paths being published.
- `quartz.config.ts` owns site metadata, URL base, theme, parser/transformer/filter/emitter pipeline. `quartz.layout.ts` composes shared and page-specific Quartz components. Make site-level behavior changes there rather than editing generated output.

## Architecture

Quartz processes content through configured transformers (Markdown and HTML transformations), filters, and emitters. Emitters produce content pages and supporting indexes/assets; the layout selects components that render each page. The root config and layout are copied into the Quartz submodule before the build, so changes to either take effect in deployment. The GitHub Actions workflow builds on pushes and pull requests to `main`, and deploys successful `main` builds to GitHub Pages.
