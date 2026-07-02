---
title: "Org-roam MCP Server"
date: 2025-09-19
draft: false
---

## Goal {#goal}

Connect your local [org roam]({{< relref "20230220220100-org_roam.md" >}}) knowledge base to [Codex CLI]({{< relref "2025-10-09-180435-coding_agent.md#codex-cli" >}}) via an `org-roam-mcp` [Message Control Protocol (MCP)]({{< relref "2025-04-16-081127-ai_agent_protocols.md#message-control-protocol--mcp" >}}) server, so Codex can search, read, and write notes using MCP tools.


## Important: there are two different "org-roam-mcp" servers {#important-there-are-two-different-org-roam-mcp-servers}

The name `org-roam-mcp` is currently used by different projects. The setup depends on which one you are running.


### A) Emacs-backed (`dcruver/org-roam-ai`) {#a-emacs-backed--dcruver-org-roam-ai}

-   This MCP server talks to a running Emacs via `emacsclient` (NOT directly to SQLite).
-   It has both HTTP and stdio modes; for Codex CLI you must pass `--stdio`.
-   Repo: <https://github.com/dcruver/org-roam-ai>


#### Which install source are you using? {#which-install-source-are-you-using}

There are (at least) two common ways to run it, and **the required Emacs packages differ**:

<!--list-separator-->

-  Recommended: run the latest server from GitHub (matches `org-roam-second-brain`)

    -   Run with uvx:
        ```sh
        uvx --from git+https://github.com/yanboyang713/org-roam-ai.git#subdirectory=mcp org-roam-mcp --stdio
        ```
    -   Emacs must have these features loaded:
        -   `org-roam-vector-search` (from `org-roam-second-brain`)
        -   `org-roam-second-brain` (from `org-roam-second-brain`)
        -   `org-roam-api` (from `org-roam-ai`)

<!--list-separator-->

-  Legacy: PyPI (`uvx org-roam-mcp`) (older checks + different defaults)

    -   Run with uvx:
        ```sh
        uvx org-roam-mcp --stdio
        ```
    -   Default server file path is often `~/emacs-server/server` (override via `EMACS_SERVER_FILE`).
    -   Emacs is checked for these loaded features:
        -   `org-roam-vector-search`
        -   `org-roam-ai-assistant`
        -   `org-roam-api`
    -   Details set-up steps: [org-roam-ai (Emacs-backed server; Codex via `--stdio`)](#org-roam-ai--emacs-backed-server-works-with-codex-via-stdio)


### B) SQLite-backed (GitHub `aserranoni/org-roam-mcp`) {#b-sqlite-backed--github-aserranoni-org-roam-mcp}

-   Talks directly to the org-roam SQLite DB (`org-roam.db`) and org files.
-   Does NOT require Emacs server or extra elisp packages.
-   Repo: <https://github.com/aserranoni/org-roam-mcp>


## Comparison — two different "org-roam-mcp" servers {#comparison-two-different-org-roam-mcp-servers}

| Topic                       | dcruver/org-roam-ai (PyPI org-roam-mcp)                                                              | aserranoni/org-roam-mcp (GitHub)                                       |
|-----------------------------|------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| Primary goal                | Semantic search + AI-assisted workflows                                                              | Direct org-roam DB/file access via SQLite                              |
| Backend                     | Emacs via `emacsclient` + elisp APIs                                                                 | SQLite (`org-roam.db`) + org files                                     |
| Needs Emacs server?         | Yes (`M-x server-start` or `emacs --daemon`)                                                         | No                                                                     |
| Needs extra Emacs packages? | Yes: `org-roam-api` plus semantic packages (`org-roam-vector-search`, `org-roam-second-brain`, etc.) | No                                                                     |
| Needs embeddings service?   | Yes (Ollama / Infinity / any OpenAI-compatible embeddings endpoint)                                  | No                                                                     |
| Stdio mode for Codex        | Yes (must pass `--stdio`)                                                                            | Yes (stdio server)                                                     |
| Typical tools               | `semantic_search`, `contextual_search`, `create_note`, `generate_embeddings`, daily note helpers     | `search_nodes`, `get_node`, `get_backlinks`, `create_node`, `add_link` |
| Best for                    | RAG/semantic retrieval, “AI assistant” workflows inside org-roam                                     | Lightweight “query my org-roam DB/files” integration                   |


## Prereqs {#prereqs}

1.  [Codex CLI]({{< relref "2025-10-09-180435-coding_agent.md#codex-cli" >}}) installed and logged in
    ```sh
    codex --version
    codex login
    ```
2.  **Org-roam database exists and is up-to-date**
    -   In Emacs:
        -   Run `M-x org-roam-db-sync`
        -   (Recommended) Enable `org-roam-db-autosync-mode` so the DB stays current
3.  **Python runner for the server**
    -   Recommended: `uv` (so you can run the server with `uvx`)
    -   Alternative: `pip` in a virtualenv


## org-roam-ai (Emacs-backed server, works with Codex via `--stdio`) {#org-roam-ai--emacs-backed-server-works-with-codex-via-stdio}

This section fixes the common errors:

-   `emacsclient: error accessing server file ...`
-   `Required Emacs packages not loaded: ...`


### Start Emacs server (create the server file) {#start-emacs-server--create-the-server-file}

Start Emacs server once:

-   In Emacs: `M-x server-start`
-   Or from shell:
    ```sh
    emacs --daemon
    ```

Important (why this matters):

-   The MCP server (`org-roam-mcp`) calls Emacs like this:
    -   `emacsclient --server-file=/path/to/server -e '(...)'`
-   The `--server-file` option uses a **TCP auth file** (not a UNIX socket).
-   So your Emacs server must run with:
    -   `server-use-tcp` enabled
    -   a readable/writable, secure `server-auth-dir` (permissions 700)

If daemon startup fails because Emacs can’t write to `/run/user/...` (common in containers), or you see “can't find socket”, force TCP mode and use a safe auth dir (example):

```emacs-lisp
(with-eval-after-load 'server
  ;; org-roam-mcp calls `emacsclient --server-file=...` (TCP auth file),
  ;; so force the Emacs server to use TCP and bind locally.
  (setq server-use-tcp t
        server-host "127.0.0.1")

  ;; Ensure the auth dir exists and is secure (Emacs requires 700 permissions).
  (setq server-auth-dir (expand-file-name "server/" user-emacs-directory))
  (unless (file-directory-p server-auth-dir)
    (make-directory server-auth-dir t))
  (set-file-modes server-auth-dir #o700)

  (unless (server-running-p) (server-start)))
```

Where to put this (example for a vanilla config):

-   Put the TCP/auth-dir settings in `~/.emacs.d/early-init.el` (loads very early)
-   Ensure the server is started in `~/.emacs.d/init.el`, e.g.:
    ```emacs-lisp
    (require 'server)
    (unless (server-running-p) (server-start))
    ```

Then ensure the auth dir is private (otherwise Emacs refuses to start the server):

```sh
chmod 700 ~/.emacs.d/server
```

Then confirm the server file exists:

```sh
ls -la ~/.emacs.d/server/server 2>/dev/null
```

Then confirm you can connect using `--server-file`:

```sh
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(emacs-version)'
```


### Install + load the required Emacs packages {#install-plus-load-the-required-emacs-packages}

The recommended setup (current upstream) uses:

-   `org-roam-second-brain` (provides `org-roam-second-brain` and `org-roam-vector-search`)
-   `org-roam-api` (from `org-roam-ai`)

Example setup for `straight.el`:

```emacs-lisp
(straight-use-package
 '(org-roam-second-brain
   :type git
   :host github
   :repo "dcruver/org-roam-second-brain"))

(straight-use-package
  '(org-roam-api
    :type git
    :host github
    :repo "dcruver/org-roam-ai"
    :files ("packages/org-roam-ai/org-roam-api.el")))

;; Ensure the features are actually loaded (required for org-roam-mcp startup)
(require 'org-roam-vector-search)
(require 'org-roam-second-brain)
(require 'org-roam-api)

;; Configure embeddings (required for semantic features)
;; Ollama example:
;;   ollama pull nomic-embed-text
;;   (setq org-roam-semantic-embedding-url "http://localhost:11434/v1"
;;         org-roam-semantic-embedding-model "nomic-embed-text")
(setq org-roam-semantic-embedding-url "http://localhost:11434/v1"
      org-roam-semantic-embedding-model "nomic-embed-text")
```

Optional (recommended): generate embeddings once so `semantic_search` works well:

-   In Emacs: `M-x org-roam-semantic-generate-all-embeddings`
-   Or via MCP (after Codex is connected): call the `generate_embeddings` tool

Optional: org-roam-second-brain “second brain” features (digest, stale projects, etc.)

-   Try: `M-x sb/digest` (keybindings are documented in <https://github.com/dcruver/org-roam-second-brain>)


#### Create predefined structured notes (Person / Project / Idea / Admin) {#create-predefined-structured-notes--person-project-idea-admin}

The package creates these notes directly (it does **not** use org-roam capture templates).

How to create them:

-   Run interactively:
    -   `M-x sb/person` (prompts for name + context)
    -   `M-x sb/project` (prompts for title + next action)
    -   `M-x sb/idea` (prompts for title + one-liner)
    -   `M-x sb/admin` (prompts for task + due date)
-   Or use the default keybindings (global `C-c b` prefix):
    -   `C-c b p` → person
    -   `C-c b P` → project
    -   `C-c b i` → idea
    -   `C-c b a` → admin

Where the files go:

-   Under your `org-roam-directory` (check in Emacs: `M-x describe-variable RET org-roam-directory`)
-   Default subfolders:
    -   people/, projects/, ideas/, admin/
-   Customize the folders via `M-x customize-group RET sb RET` (variable `sb/directories`).

What’s inside the files:

-   A properties drawer including:
    -   `:ID:` (org id)
    -   `:NODE-TYPE:` person/project/idea/admin
    -   extra fields depending on type (e.g. `:CONTEXT:`, `:LAST-CONTACT:`, `:STATUS:`, `:NEXT-ACTION:`, `:ONE-LINER:`, `:DUE-DATE:`)
-   Then headings like:
    -   Person: “\* Follow-ups” and “\* Notes”
    -   Project: “\* Next Actions” and “\* Notes”
    -   Idea: “\* One-liner” and “\* Elaboration”
    -   Admin: “\* Notes”

Related helper commands:

-   `M-x sb/inbox` (or `C-c b I`): logs text to today’s daily note and auto-creates person nodes for any `[[Name]]` links
-   `M-x sb/dangling` (or `C-c b u`): shows dangling `[[Name]]` links you might want to convert into person nodes

Verify Emacs has them loaded:

```sh
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-api))'
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-second-brain))'
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-vector-search))'
```


### Step-by-step (end-to-end, tested) {#step-by-step--end-to-end-tested}

This is the minimal, repeatable sequence that makes the MCP server start successfully.


#### 1) Start (or restart) the Emacs daemon {#1-start--or-restart--the-emacs-daemon}

```sh
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(kill-emacs)' 2>/dev/null || true
emacs --daemon
```


#### 2) Verify the daemon is reachable via TCP auth file {#2-verify-the-daemon-is-reachable-via-tcp-auth-file}

```sh
ls -la ~/.emacs.d/server/server
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(emacs-version)'
```


#### 3) Verify required features are loaded {#3-verify-required-features-are-loaded}

```sh
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-vector-search))'
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-second-brain))'
emacsclient --server-file=$HOME/.emacs.d/server/server -e '(featurep (quote org-roam-api))'
```


#### 4) Run the MCP server (stdio mode) {#4-run-the-mcp-server--stdio-mode}

Upstream:

```sh
EMACS_SERVER_FILE=$HOME/.emacs.d/server/server \
  uvx --from git+https://github.com/dcruver/org-roam-ai.git#subdirectory=mcp org-roam-mcp --stdio
```

My fork (replace with your repo if needed):

```sh
EMACS_SERVER_FILE=$HOME/.emacs.d/server/server \
  uvx --from git+https://github.com/yanboyang713/org-roam-ai.git#subdirectory=mcp org-roam-mcp --stdio
```

You should see logs like:

-   “All required Emacs packages are loaded”
-   “Starting org-roam MCP server in stdio mode...”


#### 5) Register the MCP server in Codex CLI {#5-register-the-mcp-server-in-codex-cli}

Upstream:

```sh
codex mcp add org-roam \
  --env EMACS_SERVER_FILE=$HOME/.emacs.d/server/server \
  -- uvx --from git+https://github.com/dcruver/org-roam-ai.git#subdirectory=mcp org-roam-mcp --stdio
```

My fork:

```sh
codex mcp add org-roam \
  --env EMACS_SERVER_FILE=$HOME/.emacs.d/server/server \
  -- uvx --from git+https://github.com/yanboyang713/org-roam-ai.git#subdirectory=mcp org-roam-mcp --stdio
```

Verify:

```sh
codex mcp list
codex mcp get org-roam
```


### Troubleshooting (org-roam-ai MCP server) {#troubleshooting--org-roam-ai-mcp-server}


#### “emacsclient: can't find socket; have you started the server?” {#emacsclient-can-t-find-socket-have-you-started-the-server}

-   You’re connecting without `--server-file`. Use:
    ```sh
    emacsclient --server-file=$HOME/.emacs.d/server/server -e '(emacs-version)'
    ```
-   If the file doesn’t exist, start the daemon:
    ```sh
    emacs --daemon
    ```


#### “emacsclient: error accessing server file …” {#emacsclient-error-accessing-server-file}

-   Usually means Emacs is not running in TCP mode (`server-use-tcp`), or you’re pointing at the wrong file.
-   Ensure Emacs has:
    -   `server-use-tcp t`
    -   `server-host "127.0.0.1"`
    -   `server-auth-dir "~/.emacs.d/server/"`


#### “... is not a safe directory because it is accessible by others (755)” {#dot-dot-dot-is-not-a-safe-directory-because-it-is-accessible-by-others--755}

-   Fix permissions:
    ```sh
    chmod 700 ~/.emacs.d/server
    ```


#### “org-roam-ui … Cannot bind server socket: Address already in use” {#org-roam-ui-cannot-bind-server-socket-address-already-in-use}

-   That’s org-roam-ui’s webserver port being busy.
-   Fix: don’t auto-enable org-roam-ui on startup; enable it manually when needed (`M-x org-roam-ui-mode`), or change the port.
-   Example (only enable automatically in GUI, and don’t fail startup):
    ```emacs-lisp
    (use-package org-roam-ui
      :after org-roam
      :config
      (when (display-graphic-p)
        (ignore-errors (org-roam-ui-mode 1))))
    ```


## aserranoni/org-roam-mcp (SQLite-backed) {#aserranoni-org-roam-mcp--sqlite-backed}


### Step 1 — Identify your org-roam DB and directory (SQLite-backed server only) {#step-1-identify-your-org-roam-db-and-directory--sqlite-backed-server-only}

The MCP server needs two things:

-   The org-roam SQLite DB file (often named `org-roam.db`)
-   The org-roam directory containing your roam `.org` files


#### Find the DB file {#find-the-db-file}

In my case: ~/.emacs.d/org-roam.db

Check common DB locations:

```sh
ls -la ~/.emacs.d/org-roam.db ~/.config/emacs/org-roam.db ~/org-roam.db 2>/dev/null
```

If you don’t see it, find it:

```sh
find ~ -maxdepth 4 -name "org-roam.db" 2>/dev/null
```


#### Find the org-roam directory {#find-the-org-roam-directory}

It is whatever you configured as `org-roam-directory` in Emacs.
Common locations: `~/org-roam/`, `~/Documents/org-roam/`, `~/Dropbox/org-roam/`
In my case: ~/org/org-roam/references

Quick sanity check (must contain your roam org files):

```sh
ls -la ~/org-roam 2>/dev/null | head
```


### Step 2 — Run/Install the org-roam MCP server (SQLite-backed server only) {#step-2-run-install-the-org-roam-mcp-server--sqlite-backed-server-only}

For the SQLite-backed server, install/run directly from GitHub to avoid name collisions with the PyPI `org-roam-mcp` (which is the Emacs-backed server).


#### Option A (recommended) — Run with uvx (no install) {#option-a--recommended--run-with-uvx--no-install}

```sh
uvx --from git+https://github.com/aserranoni/org-roam-mcp org-roam-mcp
```


#### Option B — Install with pip in a virtualenv {#option-b-install-with-pip-in-a-virtualenv}

```sh
python -m venv ~/.venvs/org-roam-mcp
source ~/.venvs/org-roam-mcp/bin/activate
pip install --upgrade pip
pip install git+https://github.com/aserranoni/org-roam-mcp
org-roam-mcp
```


### Step 3 — Register the MCP server in Codex CLI (SQLite-backed server only) {#step-3-register-the-mcp-server-in-codex-cli--sqlite-backed-server-only}

Codex can manage MCP servers via `codex mcp`.


#### Add the server (auto-detect mode) {#add-the-server--auto-detect-mode}

Try this first (the server attempts to auto-detect DB + directory):

```sh
codex mcp add org-roam -- uvx --from git+https://github.com/aserranoni/org-roam-mcp org-roam-mcp
```


#### Add the server (explicit paths; recommended if auto-detect fails) {#add-the-server--explicit-paths-recommended-if-auto-detect-fails}

Replace the two paths with your real values:

```sh
codex mcp add org-roam \
  --env ORG_ROAM_DB_PATH=/path/to/org-roam.db \
  --env ORG_ROAM_DIR=/path/to/org-roam/ \
  -- uvx --from git+https://github.com/aserranoni/org-roam-mcp org-roam-mcp
```

Optional: increase result limit (default is usually 50):

```sh
codex mcp add org-roam \
  --env ORG_ROAM_DB_PATH=/path/to/org-roam.db \
  --env ORG_ROAM_DIR=/path/to/org-roam/ \
  --env ORG_ROAM_MAX_SEARCH_RESULTS=200 \
  -- uvx --from git+https://github.com/aserranoni/org-roam-mcp org-roam-mcp
```


#### Verify Codex sees the server {#verify-codex-sees-the-server}

```sh
codex mcp list
codex mcp get org-roam
```


### Step 4 — Use it in Codex {#step-4-use-it-in-codex}

Start Codex normally:

```sh
codex
```

Then prompt it to use the org-roam MCP tools. Examples:

-   “Use the org-roam MCP tools to search for nodes about ‘kubernetes’.”
-   “Show backlinks for my note titled ‘Deep Learning’.”
-   “Create a new org-roam note titled ‘MCP ideas’ with tags mcp and codex.”


### Troubleshooting {#troubleshooting}


#### DB not found / empty results {#db-not-found-empty-results}

-   Run `M-x org-roam-db-sync` in Emacs, then retry.
-   Ensure `ORG_ROAM_DB_PATH` points to the correct `org-roam.db`.
-   Ensure `ORG_ROAM_DIR` is the directory that contains your roam `.org` files (not a parent folder).


#### DB is stale after creating/updating notes {#db-is-stale-after-creating-updating-notes}

-   Ensure org-roam auto-sync is enabled (`org-roam-db-autosync-mode`).
-   If needed, manually run `M-x org-roam-db-sync` after file changes.


#### Codex doesn’t list the server {#codex-doesn-t-list-the-server}

-   Re-run:
    ```sh
    codex mcp list
    codex mcp get org-roam
    ```
-   If you need to re-add:
    ```sh
    codex mcp remove org-roam
    ```

then re-run the `codex mcp add` command.


## Notes {#notes}

-   The MCP server runs locally; access is defined by the DB + directory paths you give it.
-   If you have multiple org-roam databases (work/personal), consider adding multiple servers:
    -   `org-roam-personal`, `org-roam-work` with different `ORG_ROAM_DB_PATH` and `ORG_ROAM_DIR`.


## Reference List {#reference-list}

-   dcruver/org-roam-ai (Emacs-backed PyPI server): <https://github.com/dcruver/org-roam-ai>
-   aserranoni/org-roam-mcp (SQLite-backed): <https://github.com/aserranoni/org-roam-mcp>
-   LobeHub listing: <https://lobehub.com/bg/mcp/aserranoni-org-roam-mcp>
