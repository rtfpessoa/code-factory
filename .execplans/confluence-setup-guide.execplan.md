# Create Opinionated Claude Code Setup Guide on Confluence

This ExecPlan is a living document. The sections Progress, Surprises & Discoveries, Decision Log, and Outcomes & Retrospective must be kept up to date as work proceeds.


## Purpose / Big Picture

After this work is complete, a new Confluence page titled "Opinionated Claude Code Setup Guide" will exist in Rodrigo Fernandes' personal space on datadoghq.atlassian.net. The page is a comprehensive, opinionated guide -- written in Rodrigo's voice and first-person perspective -- that explains how to set up Claude Code in a way that centralizes all configuration in a single Git repository (the "code-factory" repo at github.com/rtfpessoa/code-factory) and symlinks it into the home directory. The guide covers installation, settings, plugins, MCP servers, the OpenCode CLI companion setup, and the code-factory approach itself. It is modeled after Mat Brown's guide (page ID 6192955802 in space ~614b8bcacee20b0069e83bd0) but adapted to Rodrigo's actual configuration, which is more extensive and follows a different philosophy (git-managed config files rather than manual editing).

Anyone reading the guide should be able to replicate Rodrigo's Claude Code environment from scratch by cloning the code-factory repo and running the bootstrap script.


## Progress

- [ ] Draft the full Markdown body of the Confluence page.
- [ ] Call the Atlassian MCP tool createConfluencePage to publish it.
- [ ] Verify the page was created successfully by reading it back.


## Surprises & Discoveries

(None yet.)


## Decision Log

- Decision: Use the Atlassian MCP createConfluencePage tool with contentFormat "markdown" rather than ADF.
  Rationale: The task explicitly requests Markdown format. Confluence's API accepts Markdown and renders it to its internal storage format on creation, which is the simplest path. ADF would require manually constructing a JSON document tree, which is error-prone and unnecessary.
  Date/Author: 2026-02-13 / Rodrigo (plan author)

- Decision: Structure the guide with these top-level sections: Philosophy (the code-factory approach), Install Claude Code, Settings, Plugins, MCP Servers, OpenCode CLI, and Appendix.
  Rationale: Mat Brown's guide has four sections (Install, Settings, Plugins, Appendix). Rodrigo's setup is more complex and introduces concepts Mat's guide does not cover (code-factory repo, MCP servers, OpenCode CLI, multi-provider agent profiles). Adding dedicated sections for these unique aspects keeps the guide clear and navigable. The "Philosophy" section goes first because the centralized-config approach is the most distinctive aspect and affects how every other section is understood.
  Date/Author: 2026-02-13 / Rodrigo (plan author)

- Decision: Include actual code snippets from the real config files verbatim.
  Rationale: The task requires including real config snippets. Readers should be able to copy-paste or diff against their own setups. The snippets are already public in the code-factory GitHub repo.
  Date/Author: 2026-02-13 / Rodrigo (plan author)

- Decision: Reference Mat Brown's guide by name and link at the top.
  Rationale: The task asks to reference Mat Brown's guide as inspiration. Giving credit and linking to it lets readers compare approaches.
  Date/Author: 2026-02-13 / Rodrigo (plan author)

- Decision: Create the content as a regular Confluence page rather than a blog post, with manual conversion afterward.
  Rationale: The user wants a blog post, but the Atlassian MCP server's createConfluencePage tool only supports creating regular pages -- there is no createConfluenceBlogPost tool or blog-type parameter available. The workaround is to create a regular page first using the MCP tool, then have the user manually convert it to a blog post via the Confluence UI (click the "..." menu on the page, then select "Move to blog"). This is a one-time manual step that takes seconds and avoids the need for raw API calls outside the MCP toolset.
  Date/Author: 2026-02-13 / Review feedback from user


## Outcomes & Retrospective

(To be filled after execution.)


## Context and Orientation

This task does not modify any code in the code-factory repository. It creates an external artifact -- a Confluence page -- using the Atlassian MCP server that is already configured and enabled in this environment.

Key files in the code-factory repo (all at the repository root, path: /Users/rodrigo.fernandes/dev/code-factory/):

- settings.json -- Claude Code global settings (symlinked to ~/.claude/settings.json by init.sh). Contains environment variables, permissions allow-list, plugin configuration, marketplace definitions, MCP server toggles, model selection, status line config, and the alwaysThinkingEnabled flag.
- mcp.json -- MCP server definitions (symlinked to ~/.mcp.json). Defines three servers: Atlassian (SSE, enabled), Datadog (HTTP, disabled), and chrome-devtools (local npx command).
- opencode.jsonc -- OpenCode CLI configuration (symlinked to ~/.config/opencode/opencode.jsonc). Defines providers (Anthropic, Google, OpenAI), model settings with thinking budgets, named agent profiles for A/B testing (codex, opus, gemini), MCP server config mirroring Claude Code's, and granular permission patterns.
- init.sh -- Bootstrap script that creates symlinks from the three files above into the user's home directory. Idempotent: re-running it replaces existing symlinks and skips regular files with a warning.
- Makefile -- Validation system with targets: all, check (frontmatter, agents, refs, structure), lint (JSON/JSONC).

The code-factory repo is also a Claude Code plugin marketplace containing three plugins (productivity, git, code) with 12 skills total covering commit workflows, branch management, PR creation, code review, documentation, and more.

The Atlassian MCP tool mcp__atlassian__createConfluencePage is available and requires: cloudId, spaceId, body, title, and contentFormat. The cloudId is "datadoghq.atlassian.net", the spaceId is "2165375258", and the contentFormat is "markdown".

Mat Brown's original guide (page ID 6192955802) is in a different space and serves as the structural and tonal reference. It is written in a friendly, casual first-person voice with code blocks, tables, and bullet lists. The guide has four sections: Install Claude Code, Settings (release channel, environment variables, permissions, status line), Plugins (Anthropic official, Datadog marketplace, Mat's personal marketplace), and Appendix (IDE opinion about Zed).


## Plan of Work

There is a single milestone: compose the Markdown content for the Confluence page and publish it using the MCP tool.

The Markdown body must include the following sections, each described in detail below.

### Section: Introduction

A short paragraph in first person explaining that this guide describes Rodrigo's Claude Code setup, that it is opinionated, and that it was inspired by Mat Brown's guide (link to the original page). Mention that the key difference is that everything lives in a Git repo and gets symlinked into place.

### Section: The code-factory Approach

Explain the philosophy: instead of manually editing ~/.claude/settings.json and ~/.mcp.json, all config files live in a Git repo (github.com/rtfpessoa/code-factory). A bootstrap script (init.sh) creates symlinks. This means the setup is version-controlled, reproducible across machines, and easy to share. Include the init.sh script content. Mention that the repo also doubles as a Claude Code plugin marketplace with three plugins (productivity, git, code) containing 12 skills.

### Section: Install Claude Code

Same as Mat's: the curl install command, plus a link to the internal Datadog setup page.

### Section: Settings

Cover these subsettings:

1. Release channel -- stable, same rationale as Mat's guide.
2. Environment variables -- table of ENABLE_TOOL_SEARCH, CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY, DISABLE_NON_ESSENTIAL_MODEL_CALLS. Note what each does. Explain why ENABLE_TOOL_SEARCH is included (it enables deferred tool loading for MCP servers with many tools, improving startup time and reducing noise). Note that unlike Mat's guide, CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR and USE_BUILTIN_RIPGREP are not used here.
3. Model and thinking -- default model is opus, alwaysThinkingEnabled is true.
4. Permissions -- the full allow list with commentary organizing them by category (read-only shell, Datadog-specific tools, git operations, gh CLI, built-in tools, MCP tools). Include the additionalDirectories list.
5. Status line -- ccstatusline, same as Mat's.

### Section: Plugins

Organized by marketplace:

1. Anthropic official (claude-plugins-official) -- 7 plugins: code-review, code-simplifier, superpowers, commit-commands, pr-review-toolkit, gopls-lsp, rust-analyzer-lsp. Brief description of each.
2. Datadog marketplace (datadog-claude-plugins) -- 5 plugins: dual-agents-review, osx-notifications, skills, dd, writing.
3. Mat Brown's marketplace (mat-brown-contrib) -- 5 plugins: common-mistakes, git-mergetool, git-split-branch, permissions, pr-description.
4. code-factory marketplace -- 3 plugins (productivity, git, code) that are part of the repo itself. Mention the 12 skills they provide.

Include the full enabledPlugins and extraKnownMarketplaces JSON blocks.

### Section: MCP Servers

Cover the three servers defined in mcp.json:
1. Atlassian (enabled) -- SSE connection for Jira/Confluence integration.
2. Datadog (disabled) -- HTTP connection for core/software-delivery/error-tracking toolsets. Explain it is disabled by default because the toolset is still under development.
3. Chrome DevTools -- local npx server for browser debugging.

Include the mcp.json content and the enabledMcpjsonServers/disabledMcpjsonServers settings.

### Section: OpenCode CLI

Explain that alongside Claude Code, the setup also configures OpenCode CLI (opencode.ai), a terminal-based coding agent that supports multiple LLM providers. Cover:
- Multi-provider setup (Anthropic, Google, OpenAI) with model-specific tuning.
- Named agent profiles (codex, opus, gemini) for easy A/B comparison.
- Shared MCP server config.
- Granular permission system with secret-file denies.
- The fact that opencode.jsonc is also symlinked by init.sh.

### Section: Makefile Validation (brief)

Mention that the code-factory repo includes a Makefile with validation targets (make all = check + lint) that ensure plugin metadata, skill cross-references, and JSON files remain valid. This is useful when developing custom plugins.

### Section: Closing

A short paragraph encouraging readers to adapt the setup to their needs and linking to the code-factory repo.


## Concrete Steps

All commands are run from the working directory /Users/rodrigo.fernandes/dev/code-factory.

Step 1: Compose the full Markdown body as a string. This is done in code, not as a shell command. The content follows the section plan described above.

Step 2: Call the MCP tool mcp__atlassian__createConfluencePage with these parameters:
- cloudId: "datadoghq.atlassian.net"
- spaceId: "2165375258"
- title: "Opinionated Claude Code Setup Guide"
- contentFormat: "markdown"
- body: (the composed Markdown string)

Expected result: The tool returns a JSON object containing the new page's id, title, status ("current"), and a _links object with a webui URL.

Step 3: Verify by calling mcp__atlassian__getConfluencePage with the returned page ID to confirm the content was stored correctly.

Expected result: The page body matches the submitted Markdown (modulo Confluence's internal rendering).

Step 4 (manual, user action): Convert the page to a blog post. The Atlassian MCP tool can only create regular pages, not blog posts. After confirming the page content is correct in Step 3, open the page in the Confluence UI, click the "..." (more actions) menu in the top-right corner, and select "Move to blog". This converts the page into a blog post in Rodrigo's personal space. The conversion is instantaneous. Note that the page URL will change after this step -- the new URL will contain "/blog/" instead of "/pages/" and the old URL will redirect.


## Validation and Acceptance

Success is defined as: the Confluence page exists at datadoghq.atlassian.net in space ~642481942 (space ID 2165375258), is titled "Opinionated Claude Code Setup Guide", and contains all the sections described in the Plan of Work. A human visiting the page URL should see a well-formatted guide with code blocks, tables, and organized sections that reflect Rodrigo's actual configuration files.

To verify programmatically after creation:
1. Read the page back using mcp__atlassian__getConfluencePage with the page ID returned by the create call.
2. Confirm the title is "Opinionated Claude Code Setup Guide".
3. Confirm the body contains key strings: "code-factory", "init.sh", "ENABLE_TOOL_SEARCH", "settings.json", "mcp.json", "opencode.jsonc", "ccstatusline", and plugin names from the enabledPlugins list.

After the user manually converts the page to a blog post (Step 4 in Concrete Steps), the page URL will change. The original URL returned by the createConfluencePage tool will redirect to the new blog post URL. The page ID remains the same, so mcp__atlassian__getConfluencePage will still work with the original ID, but the webui link in its response will reflect the new blog URL path.


## Idempotence and Recovery

The createConfluencePage tool creates a new page each time it is called. If the step is run multiple times, duplicate pages will be created. To recover: use the Confluence UI or the updateConfluencePage / delete API to remove duplicates. The page ID returned by the first successful call should be noted so that subsequent runs can use updateConfluencePage instead of create.

If the create call fails (e.g., authentication error, network issue), it is safe to retry. No partial page is left behind on failure.


## Artifacts and Notes

The Markdown body to be submitted is the primary artifact. It is composed entirely from the config files already read from the code-factory repo and from Mat Brown's guide structure. No external resources need to be fetched at execution time.

Key source files and their paths within the repo (relative to /Users/rodrigo.fernandes/dev/code-factory/):

    settings.json   -- Claude Code settings (179 lines)
    mcp.json        -- MCP server config (16 lines)
    opencode.jsonc  -- OpenCode CLI config (309 lines)
    init.sh         -- Bootstrap script (54 lines)
    Makefile        -- Validation targets (183 lines)

The Confluence page content in Markdown follows. This is the exact body that will be passed to the createConfluencePage tool:

(See the body composed during execution in the Concrete Steps milestone.)


## Interfaces and Dependencies

External service: Atlassian Confluence via MCP server (already configured and enabled in mcp.json and settings.json).

MCP tools used:
- mcp__atlassian__createConfluencePage -- creates the page.
- mcp__atlassian__getConfluencePage -- verifies the page after creation.

No code changes are made to the repository. No commits are created. The .execplans/ directory and this file are working documents that are never committed.

Confluence API parameters:
- cloudId: "datadoghq.atlassian.net"
- spaceId: "2165375258" (Rodrigo's personal space, space key ~642481942)
- contentFormat: "markdown"

The page is created at the top level of the personal space (no parentId specified).
