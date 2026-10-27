# NightCoop on Bazzite (Linux) — install guide

NightCoop is an unofficial co-op mod for Cyberpunk 2077. Its site only documents
Windows. On Linux it runs fine under Proton, but two things break out of the box.
The script in this folder fixes both and installs the mod.

## What the script does

1. Finds your Steam library, the Cyberpunk 2077 folder, and the game's Proton prefix.
2. Checks the game version and picks the matching NightCoop build from nightcoop.net,
   verifying its SHA-256 against the site's own release list.
3. Backs up your saves and every file it is about to replace into
   `~/nightcoop-backup-<date>/`.
4. Installs NightCoop, then overlays **RED4ext 1.31** on top of the 1.30 the archive
   bundles. 1.30 does not work on game patch 2.5.
5. Installs the **Visual C++ 2022 runtime** into the game's Proton prefix. Proton ships
   a 2020 copy, and RED4ext crashes in it with
   *"RED4ext (error 998) No access to memory location"*. It uses the installer Steam
   already keeps in `Steamworks Shared` when present, otherwise it downloads it from
   Microsoft.
6. Sets the Steam launch options to `WINEDLLOVERRIDES="winmm=n,b" %command%`, or
   tells you to if Steam is open. Without this Proton ignores RED4ext's loader.

## Before you run it

- Own Cyberpunk 2077 on Steam and have it installed with the native Steam client
  (the one Bazzite ships, not the Flatpak).
- Launch the game once through Proton and reach the main menu, then quit. This
  creates the Proton prefix the script needs. Proton Experimental is what this was
  tested with.
- Quit the game. Closing Steam too lets the script write the launch options for you.

## Run it

```bash
chmod +x nightcoop-bazzite-install.sh
./nightcoop-bazzite-install.sh
```

It prints a plan and asks for confirmation before changing anything. Add `--yes`
to skip the question, `--dry-run` to only look, and `--no-save-backup` to skip the
save copy (it can be a couple of GB).

If you already downloaded the archives:

```bash
./nightcoop-bazzite-install.sh --nightcoop-zip ~/Downloads/NightCoop-0.1.1-beta.2.zip --red4ext-zip ~/Downloads/red4ext-1.31.0.zip
```

## Afterwards

Launch Cyberpunk 2077 from Steam. The main menu should show a new **Co-op**
("Совместная игра") entry. If it does not:

```bash
./nightcoop-bazzite-install.sh check
```

This prints the installed versions, whether the launch options are set, and the
tail of the RED4ext log. An **empty** RED4ext log means RED4ext crashed before it
could write, which is the runtime problem from step 5.

## Undo

```bash
./nightcoop-bazzite-install.sh restore ~/nightcoop-backup-<date>
```

puts back the loader, the `red4ext` folder and the prefix runtime files. Launch
options stay; clear them in Steam if you want.

## Good to know

- Everyone in a session needs the same NightCoop build. Only builds with the same
  protocol number can connect.
- A major Proton version upgrade rebuilds the prefix and brings the old 2020 runtime
  back. If error 998 returns after such an update, just run the script again; it
  skips what is already done.
- Steam's "Verify integrity of game files" does not remove the mod, but a game patch
  may need a newer NightCoop or RED4ext. Rerunning the script picks up the latest
  NightCoop build for your game version. RED4ext is pinned to 1.31.0 in the script;
  change `RED4EXT_VER`, `RED4EXT_URL` and `RED4EXT_SHA` at the top to move to a
  newer release.
- Back up your saves before the first session anyway. The script does, but it never
  hurts.
