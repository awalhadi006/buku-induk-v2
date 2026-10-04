# AGENTS.md

- This repository (`buku-induk-v2`) currently contains no source code, config, or docs.
- Do not assume conventions from the old `buku-induk` project — its directory has been removed.
- Once the project is scaffolded, replace this file with verified facts: dev/build/test/lint commands, entrypoint locations, and any non-obvious setup requirements. Only include what you can verify from the repo.

## Agent skills

### Issue tracker

Issues live as local markdown files under `.scratch/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default canonical labels: needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: GLOSSARY.md + docs/adr/ at repo root. See `docs/agents/domain.md`.

## Flowbite-Svelte MCP

You are able to use the Flowbite-Svelte MCP server, where you have access to comprehensive Flowbite-Svelte component documentation. Here's how to use the available tools effectively:

### Available MCP Tools:

#### 1. findComponent

Use this FIRST to discover components by name or category. Returns component information including the documentation path.
When asked about Flowbite-Svelte components, ALWAYS use this tool to locate the correct component before fetching documentation.
Example queries: 'Button', 'CardPlaceholder', 'form checkbox'

#### 2. getComponentList

Lists all available Flowbite-Svelte components with their categories.
Use this to discover what components are available or to help users explore component options.

#### 3. getComponentDoc

Retrieves full documentation content for a specific component. Accepts the component path found using findComponent.
After calling findComponent, use this tool to fetch the complete documentation including usage examples, props, and best practices.

#### 4. searchDocs

Performs full-text search across all Flowbite-Svelte documentation.
Use this when you need to find specific information that might span multiple components or when the user asks about features or patterns.
