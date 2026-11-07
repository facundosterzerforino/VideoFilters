# Install the audited source in Vivaldi

## Use the folder already on this PC

1. Open **`vivaldi://extensions`**.
2. Enable **Developer mode**.
3. Click **Load unpacked**.
4. Select this exact folder:

   ```text
   C:\Users\fsterzer\Downloads\VideoFilters\Extension
   ```

   Select `Extension`, not `VideoFilters`, `audit`, or a ZIP file. It contains `manifest.json`. No npm install, compilation, or administrator changes are needed.

5. Open the extension details and consider **Site access → On click** or **On specific sites** for your streaming sites. Start with on-click access; embedded players may require permission for another origin. Leave incognito and file access off unless needed.
6. Pin/show **Video Filters** in Vivaldi's extension toolbar/menu.
7. Refresh your video tab. Start a video, click the extension icon, and adjust brightness, contrast, saturation, etc.

Keep this folder in place: Vivaldi loads its files directly. The agent did **not** install the extension or change Vivaldi settings, following your choice to self-install.

## Using the controls

- The popup exposes all nine regular picture filters: brightness, contrast, saturation, grayscale, hue rotation, invert, sepia, blur, and opacity.
- Use the small reset button beside a filter to restore that control. Select the **default** preset to restore the normal filter/speed values.
- Use the preset add/save button to name a setup, then choose it from the preset dropdown on another video. Applying presets is manual; automatic per-site application is not implemented in this version.
- Avoid the **Shaders** checkbox on Netflix: those experimental effects cannot process DRM-protected video. The regular CSS filter sliders are separate.
- Do not run this alongside the earlier **FilterVideo** or another filter extension on the same video: they may overwrite each other's CSS filters. Disable the older one if you previously installed it; do not delete its source/fork to do so.

Read **AUDIT.md** first. No obvious malicious behavior was found, but broad host permission, browser-Sync preferences, retained last-frame URL, and two low-severity user-label HTML-rendering concerns are documented. Runtime code was left unchanged. Use trusted preset names and custom shader snippets.

## ZIP alternative

```text
C:\Users\fsterzer\Downloads\VideoFilters-1.4.1-audited.zip
```

Extract it into a permanent folder. Then Load unpacked from the folder containing `manifest.json`. Do not confuse it with the separate `VideoFilters-1.4.1-source.zip`, which is the complete source repository archive.

Extension ZIP SHA-256:

```text
18711cc5e705010ea9a2bece143b1f41c4dc804bfb6ed3d4d8c814a93e0dc834
```

## Source, rollback, and future updates

- Fork: https://github.com/facundosterzerforino/VideoFilters
- Original: https://github.com/FWeynschenk/VideoFilters
- Audited runtime commit: `e8f311e3ffe5047860ab8b3f8becbba8857f2ec1`
- Local repository: `C:\Users\fsterzer\Downloads\VideoFilters`
- `origin` points to your fork; `upstream` points to the original.

This unpacked copy does not automatically install newer upstream versions. Keep the audited ZIP as a rollback copy. Review upstream changes before updating; the audit does not apply to later commits.

To check for changes without replacing the installed files, run these commands in **Git Bash**:

```bash
cd 'C:/Users/fsterzer/Downloads/VideoFilters'
GIT_TERMINAL_PROMPT=0 GCM_INTERACTIVE=never timeout 90 git -c credential.helper= -c 'credential.helper=!gh auth git-credential' fetch upstream

git log --oneline e8f311e3ffe5047860ab8b3f8becbba8857f2ec1..upstream/master
git diff e8f311e3ffe5047860ab8b3f8becbba8857f2ec1..upstream/master -- Extension
```

An empty log means no new upstream commits relative to this reviewed snapshot. Fetching alone does not change your installation. Do not force-sync away the fork's audit documentation. After a reviewed update, click **Reload** on the Vivaldi extension card and refresh the video tab.

To remove the extension, use **Remove** on its card in `vivaldi://extensions`. The local source/fork can remain for reference.
