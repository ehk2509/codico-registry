# Codico registry

The packages that **Codico: Marketplace** lists: skills, agents, MCP servers and hooks for the [Codico](https://marketplace.visualstudio.com/items?itemName=codico.codico) VS Code extension.

This repository holds only `index.json`. Each package lives in its publisher's own public GitHub repository; the index pins every version to a **commit** and a **content hash**, so what was reviewed here is exactly what Codico installs. Codico downloads that commit, refuses it if its content hash differs, scans it, and shows the user what it adds, the commands it would run and what the scan found before installing anything.

## Publishing a package

1. In a public GitHub repository of your account (or your organization's), make a folder with a `codico-package.json`:

   ```json
   {
     "id": "your-account/release-notes",
     "version": "1.0.0",
     "description": "Writes release notes from merged pull requests",
     "skills": ["skills/release-notes"],
     "agents": ["agents/changelog-reviewer.md"],
     "mcpServers": { "gh-notes": { "command": "npx", "args": ["-y", "gh-notes-mcp@1.4.2"] } },
     "hooks": [{ "event": "afterEdit", "match": "CHANGELOG.md", "command": "node scripts/check-changelog.js {file}" }],
     "permissions": { "commands": ["npx", "node"], "network": ["api.github.com"], "env": ["GITHUB_TOKEN"] }
   }
   ```

   A skill is a folder with a `SKILL.md` (or one `.md` file), an agent a `.md` file, as in a project's `.codico` folder. The id starts with your GitHub account. Declare every command, network address and environment variable the package uses: the scan compares them with what it finds.

2. Commit and push, then check it:

   ```bash
   node ~/.vscode/extensions/codico.codico-<version>/bin/codico.js package check path/to/folder
   ```

   It validates the manifest, scans the package, and prints the entry for `index.json`, with the commit and the content hash of what GitHub serves for it.

3. Open a pull request here adding the entry (a new version is a new item in `versions`). CI downloads your commit and verifies the hash, the manifest, that the repository is yours, and the scan.

## Review and verification

- **Every pull request is reviewed by a maintainer**, who reads the package at its commit. CI must pass; a package with a *dangerous* scan finding is merged only when a maintainer has read it and labelled the pull request `risk-reviewed`.
- **`"verified": true`** is set only by maintainers, for publishers whose identity they have checked (the GitHub account or organization is who it says it is). Unverified packages are listed and installable, without the badge.
- **Pin what runs.** MCP servers started with `npx`, `uvx` and the like should name a version (`pkg@1.4.2`): otherwise what runs can change after the review, and the scan says so.
- Nothing in a package runs at install time. MCP servers and hooks ask the user before they first run.

To report a malicious or broken package, open an issue; it is removed from the index while it is looked at.
