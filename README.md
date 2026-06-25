# VaultFS

A single-file, self-contained encrypted filesystem that runs entirely in your browser. No server, no database, no dependencies beyond what's already in the file — open the HTML and you have a full file manager with a terminal, multi-select, drag-and-drop, and a custom encryption suite for taking snapshots in and out.

Everything lives in memory while the tab is open. Nothing persists automatically — you explicitly **export** an encrypted snapshot when you want to save your state, and **import** it later (in this tab, another browser, another machine) to pick up exactly where you left off.

## For?

It's useful as:

- A portable scratch filesystem you can carry around as a single HTML file
- A sandbox for testing file-manager UX patterns (multi-select, drag-move, context menus) without a backend
- A demonstration of building real cryptographic primitives from scratch in the browser

It is **not** a production storage system. See [Security](#security-notes) before trusting it with anything sensitive.

## Features

### File management
- Folders, files, tabs, breadcrumb navigation with a sibling-folder dropdown
- Multi-select — click, Ctrl/Cmd-click, Shift-click range, Ctrl/Cmd+A, arrow-key navigation
- Cut / copy / paste, drag-and-drop to move files between folders (including dropping onto breadcrumb segments)
- Right-click context menu — rename, duplicate, star, tag, delete, with multi-select-aware bulk actions
- Recursive folder size rollup shown in the file list
- Starred files and a recently-opened list in the sidebar
- Manual tagging (Work / Documents / Personal / Media / Archive / None) plus automatic tagging by file extension
- File templates on creation — Blank, Markdown, JSON, Code Snippet
- A real terminal: `ls`, `cd`, `mkdir`, `touch`, `write`, `rm`, `cat`, `find`, `tree`, `pwd`, `clear`, `help`

### Search
- Filename/path search
- Content search inside text files (`.txt`, `.md`, `.js`, `.json`, etc.) with a contextual match snippet

### File types
- Real binary file upload via drag-and-drop or the Upload button (stored as base64)
- Inline preview for images, audio, video, and PDFs
- Thumbnail icons for images directly in the file list
- A UTF-8 decode fallback preview for unrecognized file types (e.g. `.log`), with a heuristic that refuses to render genuine binary garbage as text
- Download any file back out byte-for-byte

### Encryption (export / import)
A custom three-cipher suite ("Hexagon") layers together for the encrypted snapshot you export:

- **Cascade** — a 4-sub-pass nonlinear XOR stream cipher with a key-derived S-box and feedback register
- **Vigenère+** — polyalphabetic substitution with a position-dependent shift that defeats classic frequency analysis
- **Transposition** — columnar block permutation with a fixed-per-run column count (so output size doesn't balloon with pass count)
- **Combined mode** — all three layered together, your choice of 7 / 12 / 20 passes per layer

Snapshots are gzip-compressed before encryption (skipped automatically if it wouldn't help, e.g. for already-dense binary data), then base64-encoded into a single copy-pasteable string.

Export requires a passphrase. A salt is optional but recommended — it hardens key derivation against precomputed attacks, with one click to generate a random one. Starred files and your recent-files list travel inside the same encrypted snapshot, so a full export → import cycle restores everything, not just the file tree.

**There is no recovery option.** If you lose the passphrase (and salt, if you used one), the snapshot is permanently unreadable. There's no backdoor, no "forgot password," nothing — that's the point of it being a real cipher.

## Usage

1. Open `vaultfs.html` in any modern browser (Chrome 80+, Firefox 113+, Safari 16.4+ — needed for the native compression API).
2. Use it like a file manager. Right-click for options, drag files in, open the terminal from the bottom bar if you want a CLI.
3. When you're done, click **Export**, set a passphrase (and ideally a salt), and copy the resulting string somewhere safe.
4. Next time, open the file again and click **Import**, paste the string back in with the same passphrase/salt/settings, and your filesystem is restored exactly as you left it.

No installation, no build step, no server. It's one HTML file with inline CSS and JS.

## Security notes

This is a hand-rolled cipher suite, not a peer-reviewed standard like AES-GCM. It was built as an engineering exercise and is reasonably resistant to casual inspection, but it has **not** been audited and should not be treated as cryptographically bulletproof. If you're protecting something that actually matters, use a vetted tool (age, GPG, a real password manager) instead of this.

Practical implications:
- Don't reuse a passphrase from somewhere important.
- The salt is optional specifically to lower friction for casual use — using one is still meaningfully better.
- Everything happens client-side in your browser; nothing is transmitted anywhere. The exported string is the only thing that leaves memory, and it's already encrypted before it does.

## Limitations

- No persistence between sessions other than manual export/import — closing the tab without exporting loses everything.
- Binary files are stored as base64 inside the JSON tree, which adds ~33% overhead before compression; very large files will make exports slow to encrypt (more passes = slower, by design).
- This is a demo/personal-use tool, not a sync service — there's no conflict resolution, no real multi-device sync, no server backup.

## License

Do whatever you want with it.
