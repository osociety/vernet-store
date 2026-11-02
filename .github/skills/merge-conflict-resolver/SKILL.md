---
name: merge-conflict-resolver
description: "Resolve merge conflicts when syncing upstream changes into the store repo. Handles content conflicts, modify/delete conflicts, and pubspec.lock merges while retaining both sides."
argument-hint: "PR URL, merge conflict, or 'sync fork' request"
---

# Merge Conflict Resolver

Use this when a PR (typically a "Sync fork" from `osociety/vernet` into `osociety/vernet-store`) has merge conflicts that need manual resolution.

## Procedure

1. **Fetch upstream changes**
   ```
   git fetch https://github.com/osociety/vernet.git main
   ```
   Do not add a permanent `upstream` remote unless the user asks.

2. **Attempt the merge to surface conflicts**
   ```
   git merge --no-ff FETCH_HEAD
   ```
   If it succeeds cleanly, commit and push. If it fails, proceed.

3. **Identify conflicted files**
   ```
   git diff --name-only --diff-filter=U
   ```
   Typical conflicts in this repo:
   - `lib/pages/ping_page/ping_page.dart` — content conflict (store adds ad banner import; upstream adds network score imports)
   - `pubspec.lock` — content conflict (store has webview_flutter packages; upstream has webkit_inspection_protocol)
   - `lib/pages/isp_page/isp_page_widget.dart` — modify/delete conflict (deleted upstream, modified locally)

4. **Resolve each conflict by keeping both sides**

   **Content conflicts** (`<<<<<<< HEAD` / `=======` / `>>>>>>>`):
   - Read both sides carefully.
   - Keep imports from both sides.
   - Keep features from both sides (e.g., ad banner widget AND network score snapshot saving).
   - Remove the conflict markers.

   **pubspec.lock conflicts**:
   - Keep all package entries from both sides.
   - Ensure alphabetical ordering is maintained.

   **Modify/delete conflicts**:
   - If the file was deleted upstream but modified locally, keep the local version (it's still in use by the store).
   - Stage the file with `git add`.

5. **Verify no conflict markers remain**
   ```
   grep -r "<<<<<<" lib/ pubspec.lock 2>/dev/null
   ```

6. **Stage all resolved files**
   ```
   git add <resolved-files>
   ```

7. **Complete the merge**
   ```
   git commit --no-edit
   git push origin main
   ```

## Safety

- Never use `--no-verify` or skip hooks.
- Never force-push to `main`.
- If a conflict is ambiguous (unclear which side to keep), ask the user before resolving.
- Do not delete files that exist locally just because upstream deleted them — verify they are still referenced first.
- After resolving, run `flutter analyze` if possible to catch any broken imports or references.

## Common patterns in this repo

| File | Store adds | Upstream adds | Resolution |
|------|-----------|---------------|------------|
| `ping_page.dart` | `InlineAdaptiveBanner` import + ad in list | `NetworkScoreBloc`/`NetworkScoreRepository` imports + `_savePingSnapshot` | Keep both imports, keep ad in list, keep `_savePingSnapshot` |
| `pubspec.lock` | `webview_flutter*` packages | `webkit_inspection_protocol` | Keep all packages |
| `isp_page_widget.dart` | Local modifications | File deleted | Keep local file |
