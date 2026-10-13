# TODO: Fill in the required icons

This theme is KDE Classic icons with hicolor's directory layout. It is used as
the fallback theme for GTK3 on Windows. Most files still have their old KDE
names (`fileopen`, `kcalc`, …). The freedesktop **Icon Naming Specification**
(`icon-naming-spec.xml`, v0.8.90) lists the standard names that toolkits ask
for, so the theme needs to provide those names.

## What "required" means here

- **Names:** every name in the standard tables of `icon-naming-spec.xml`
  (Actions, Animations, Applications, Categories, Devices, Emblems, MIME Types,
  Places, Status). The deprecated **Emotes** table is out of scope. The
  **International** table only lists `flag-aa` as an example of the
  `flag-<country code>` pattern, so it is optional.
- **Format** (`icon-theme-spec.xml`): `.png`, `.xpm` or `.svg`, with lower-case
  extensions. Every directory that holds icons must be listed in `Directories=`
  in `index.theme`, with a matching `[dir]` section that has a `Size=` key.
  Each image must be the pixel size of the directory it is in.
- **Sizes:** the specs don't name sizes. The targets below come from what GTK3
  asks for (menu/button 16, toolbar 16/24, DND 32, dialog 48) and from the
  `*Default`/`*Sizes` keys in `index.theme`. Every directory has
  `Type=Threshold` with the default threshold of 2, so a 22px icon also covers
  24px requests. Because of that, 24x24 is only a target for emblems.

| Context | Target sizes |
|---|---|
| actions | 16, 22, 32 |
| animations | 16, 22, 32 |
| apps | 16, 22, 32, 48, 64 |
| categories | 16, 22, 32, 48 |
| devices, mimetypes, places | 16, 22, 32, 48, 64 |
| emblems | 16, 24, 32 |
| status | 16, 22, 32, 48 (`dialog-*` needs 48) |

## Audit summary (as of 2026-10-12)

- There are 2193 PNGs across 16x16, 22x22, 24x24, 32x32, 48x48 and 64x64.
  Only about 100 of them use standard names.
- **161 standard names are missing at every size:** actions 35, apps 19,
  categories 5, devices 19, emblems 11, mimetypes 8, places 5, status 59.
  There is no `status` directory at any size.
- **724 icon files are missing** when every name is checked against its target
  sizes (✗ in the tables below).
- **598 PNGs are not the pixel size of their directory.** 110 of them have
  standard names (⚠ below). Common cases:
  - 18×18 images in `16x16/`
  - 24×24 images in `22x22/actions/`
  - 34×34 images in `32x32/` and even in `48x48/categories/`
- `folder` exists at 22px only as `22x22/actions/folder.png`, which is the
  wrong context for it.

### `index.theme` (merged with upstream `index_update.theme`)

`index.theme` now combines our old file with upstream's `index_update.theme`:
- **Directories and sections:** only upstream's plain (scale-1) directories at
  the sizes this repository uses (16, 22, 24, 32, 48 and 64). That is all
  24 upstream contexts at each size (actions … status, `stock/*`), 144 in
  total. Every directory on disk is now listed. The old typo `64x64apps` and
  the wrong `Context=Animations` on categories are gone.
- **Dropped:** upstream's `@2` (HiDPI), `scalable/*`, `symbolic/*`, 36, 72, 96,
  128, 192, 256 and 512 directories, and our old `8x8/emblems`,
  `11x11/emblems` and `scalable/categories`.
- **`[Icon Theme]` header:** upstream's `Name`, `Comment` and `Hidden`, plus
  our KDE keys (`DisplayDepth`, `*Overlay`, `*Default`/`*Sizes`).

## Plan

### Phase 0: Infrastructure
- [x] Merge `index.theme` with upstream (done). Create the missing
      directories it lists as files get added (`*/status`, `22x22/places`,
      `22x22/categories`, `22x22/devices`, `64x64/devices`, …).
- [ ] Add a check script (shell or Python) that:
      - compares the spec name tables with what is on disk,
      - flags images whose pixel size doesn't match their directory,
      - flags directories that are missing from `index.theme`.
      Use it to regenerate the tables in this file and in CI.
- [ ] Pick one method for making sizes (for example, ImageMagick
      `-filter Lanczos` followed by hand touch-up for 16 and 22) and write it
      down here. Prefer redrawing to scaling for 16 and 22.

### Phase 1: Rename/copy existing KDE icons (cheap, high value)
Many missing names already have a KDE Classic icon that fits (see "Suggested
source" below). Copy each one to its standard name and into the right context
directory. Don't rename it, because KDE apps may still use the old name. When
a source exists at 16/32/48 only, the 22 and 64 sizes are still needed (Phase 4).
- [ ] apps: all 19 have a source (`kcalc`, `konsole`, `ksysguard`, `kmix`, …).
- [ ] mimetypes: `binary`, `tgz`, `shellscript`, `vcard`, `vcalendar`, …
- [ ] devices: `media-floppy-350`, `media-optical-cd`, `kdat`, `kppp`,
      `laptop_battery`, `network`, …
- [ ] places: `kmenu`, `ftp`, `network`/`samba`, `bookmark`, …
- [ ] actions: `zoom-best-fit` (exact match), `goto`, `hmirror`/`vmirror`,
      `pagesetup`, `documentinfo`, `power`, `view-close`, …
- [ ] status: `encrypted`/`halfencrypted`/`decrypted` for `security-*`,
      `attach`, `signature`, `noatunloopplaylist`/`noatunshuffle`,
      `kdeprint_printer_*`, `password`, `error`, `info`.

### Phase 2: Normalize existing standard icons (⚠)
- [ ] Move the 24×24 images in `22x22/actions/` to a new `24x24/actions/`.
      Then make true 22×22 versions, by trimming the transparent border where
      possible.
- [ ] Trim or pad the 17–18px images in `16x16/` to 16×16 (most have a 1–2px
      shadow or margin).
- [ ] Do the same for the 34–38px images in `32x32/` and the 47px images in
      `48x48/`.
- [ ] Redo `48x48/categories/applications-graphics` and
      `applications-internet`, which are 34px images.
- [ ] Move or copy `22x22/actions/folder.png` to `22x22/places/folder.png`.

### Phase 3: Derive variants from existing art
These are overlays or small edits of icons that already exist:
- [ ] `folder-open`, `folder-drag-accept`, `folder-visiting` (from `folder`)
- [ ] `user-trash-full` (from `user-trash` or `recycled`)
- [ ] `audio-volume-{muted,low,medium,high}` (from `volume`)
- [ ] `battery-low`, `battery-caution` (from `laptop_battery`)
- [ ] `mail-read`, `mail-unread`, `mail-replied`, `mail-mark-read`,
      `mail-mark-unread` (from `message`)
- [ ] `mail-signed-verified`, `task-due`, `task-past-due`, `sync-error`,
      `system-reboot`, `system-log-out`, `edit-find-replace`,
      `document-open-recent`, `mail-send-receive`, `text-x-generic-template`,
      `drive-optical`, `format-text-direction-rtl`

### Phase 4: Fill size gaps
- [ ] Add every ✗ cell for names that exist at some size. The biggest gaps:
      - 22px for apps, devices, mimetypes, places and categories
      - 64px for everything outside mimetypes
      - 32px for a few actions
- [ ] `process-working` at 16px (a frame strip like the 22 and 32 versions).

### Phase 5: New artwork (no usable KDE source)
Draw these in KDE Classic style (flat hicolor palette, dark outline,
light from the top left):
- [ ] **status/dialogs, top priority for GTK:** `dialog-warning`,
      `image-missing`, `image-loading`
- [ ] status/network: `network-{error,idle,offline,receive,transmit,transmit-receive}`
- [ ] status/user: `user-{available,away,idle,offline}`
- [ ] status: `software-update-{available,urgent}`, `appointment-missed`
- [ ] status/weather: 11 `weather-*` icons
- [ ] emblems (16/24/32): `emblem-{default,documents,downloads,important,mail,photos,shared,synchronized,system}`
- [ ] devices: `audio-input-microphone`, `camera-web`, `input-gaming`,
      `media-flash`, `network-wireless`, `phone`, `multimedia-player`,
      `computer` (32+)
- [ ] actions: `call-start`, `call-stop`, `list-add`, `insert-link`,
      `insert-object`, `mail-mark-junk`, `mail-mark-notjunk`, `media-record`,
      `format-text-direction-ltr`
- [ ] categories: `applications-engineering`

### Phase 6: Verify
- [ ] Run the check script. It should report no missing names, no size
      mismatches and no unlisted directories.
- [ ] Run `gtk-update-icon-cache` and test with `gtk3-icon-browser` on
      Windows.
- [ ] Update `README.txt` to describe the new names and sizes.

## Detailed audit by context

Legend:
- ✓ present and the correct size
- ⚠ present but the wrong pixel size
- ✗ missing at a target size
- blank: not a target size

An icon counts as present at a size if a file with that name exists in any
context directory of that size.

### actions (target sizes: 16 22 32)

**Missing entirely (35):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `address-book-new` | ✗ | ✗ |  | ✗ |  |  | apps/kaddressbook (16,32,48) + "new" sparkle; or actions/abentry (16) |
| `call-start` | ✗ | ✗ |  | ✗ |  |  | new drawing (green handset) |
| `call-stop` | ✗ | ✗ |  | ✗ |  |  | new drawing (red handset) |
| `document-open-recent` | ✗ | ✗ |  | ✗ |  |  | derive from document-open (16,22,32) + clock overlay |
| `document-page-setup` | ✗ | ✗ |  | ✗ |  |  | actions/pagesetup (22) |
| `document-properties` | ✗ | ✗ |  | ✗ |  |  | actions/documentinfo (16) |
| `document-send` | ✗ | ✗ |  | ✗ |  |  | actions/send (22) |
| `edit-find-replace` | ✗ | ✗ |  | ✗ |  |  | derive from edit-find (16,22,32) |
| `edit-select-all` | ✗ | ✗ |  | ✗ |  |  | apps/kdat_select_all (16) |
| `find-location` | ✗ | ✗ |  | ✗ |  |  | derive from apps/kfind (16,32,48) |
| `format-text-direction-ltr` | ✗ | ✗ |  | ✗ |  |  | new drawing |
| `format-text-direction-rtl` | ✗ | ✗ |  | ✗ |  |  | mirror of format-text-direction-ltr |
| `go-jump` | ✗ | ✗ |  | ✗ |  |  | actions/goto (16,22,32) |
| `help-about` | ✗ | ✗ |  | ✗ |  |  | actions/info (16) / mimetypes/info (16,32,48) |
| `help-contents` | ✗ | ✗ |  | ✗ |  |  | actions/contents or actions/help (16,22,32) |
| `help-faq` | ✗ | ✗ |  | ✗ |  |  | actions/contexthelp (16,22,32) |
| `insert-image` | ✗ | ✗ |  | ✗ |  |  | actions/insertclipart (16) |
| `insert-link` | ✗ | ✗ |  | ✗ |  |  | new drawing (chain link) |
| `insert-object` | ✗ | ✗ |  | ✗ |  |  | new drawing |
| `insert-text` | ✗ | ✗ |  | ✗ |  |  | actions/text (16,22) |
| `list-add` | ✗ | ✗ |  | ✗ |  |  | new; pair visually with existing list-remove (16) |
| `mail-mark-important` | ✗ | ✗ |  | ✗ |  |  | actions/flag (16) |
| `mail-mark-junk` | ✗ | ✗ |  | ✗ |  |  | new drawing |
| `mail-mark-notjunk` | ✗ | ✗ |  | ✗ |  |  | new drawing |
| `mail-mark-read` | ✗ | ✗ |  | ✗ |  |  | derive from mimetypes/message (16,32,48) |
| `mail-mark-unread` | ✗ | ✗ |  | ✗ |  |  | derive from mimetypes/message (16,32,48) |
| `mail-send-receive` | ✗ | ✗ |  | ✗ |  |  | derive from mail-send (16,22,32) |
| `media-record` | ✗ | ✗ |  | ✗ |  |  | new drawing (red dot, match media-playback-* style) |
| `object-flip-horizontal` | ✗ | ✗ |  | ✗ |  |  | actions/hmirror (16) |
| `object-flip-vertical` | ✗ | ✗ |  | ✗ |  |  | actions/vmirror (16) |
| `system-log-out` | ✗ | ✗ |  | ✗ |  |  | derive from application-exit (16,22,32,48) |
| `system-reboot` | ✗ | ✗ |  | ✗ |  |  | derive from actions/power (16,32) |
| `system-shutdown` | ✗ | ✗ |  | ✗ |  |  | actions/power (16,32) |
| `window-close` | ✗ | ✗ |  | ✗ |  |  | actions/view-close (16,22,32) or dialog-close (16,22,32,48) |
| `zoom-fit-best` | ✗ | ✗ |  | ✗ |  |  | actions/zoom-best-fit (16,22) — rename/copy |

**Exist, but with size gaps or wrong pixel size (50):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `contact-new` | ⚠ | ✗ |  | ⚠ |  |  |
| `document-new` | ⚠ | ⚠ |  | ✓ |  |  |
| `document-open` | ⚠ | ⚠ |  | ✓ |  |  |
| `document-print` | ⚠ | ⚠ |  | ✓ |  |  |
| `document-print-preview` | ⚠ | ✓ |  | ✗ |  |  |
| `document-revert` | ⚠ | ✓ |  | ✓ |  |  |
| `document-save` | ⚠ | ⚠ |  | ✓ |  |  |
| `document-save-as` | ⚠ | ⚠ |  | ✗ |  |  |
| `edit-clear` | ⚠ | ✓ |  | ✓ |  |  |
| `edit-copy` | ⚠ | ✓ |  | ✓ |  |  |
| `edit-cut` | ✓ | ⚠ |  | ✓ |  |  |
| `edit-delete` | ⚠ | ✓ |  | ✓ |  |  |
| `edit-find` | ⚠ | ✓ |  | ✓ |  |  |
| `edit-paste` | ⚠ | ⚠ |  | ✓ |  |  |
| `edit-redo` | ⚠ | ✓ |  | ✓ |  |  |
| `edit-undo` | ⚠ | ✓ |  | ✓ |  |  |
| `folder-new` | ⚠ | ⚠ |  | ✓ |  |  |
| `format-indent-less` | ✗ | ✗ |  | ✓ | ✓ |  |
| `format-indent-more` | ✗ | ✗ |  | ✓ | ✓ |  |
| `format-text-strikethrough` | ✓ | ✓ |  | ✗ |  |  |
| `go-bottom` | ⚠ | ✓ |  | ✓ |  |  |
| `go-down` | ⚠ | ⚠ |  | ✓ |  |  |
| `go-home` | ⚠ | ⚠ |  | ✓ |  |  |
| `go-last` | ⚠ | ✓ |  | ✓ |  |  |
| `go-next` | ⚠ | ⚠ |  | ✓ |  |  |
| `go-previous` | ⚠ | ⚠ |  | ✓ |  |  |
| `go-top` | ⚠ | ✓ |  | ✓ |  |  |
| `go-up` | ⚠ | ⚠ |  | ✓ |  |  |
| `list-remove` | ✓ | ✗ |  | ✗ |  |  |
| `mail-forward` | ⚠ | ⚠ |  | ✓ |  |  |
| `mail-message-new` | ⚠ | ⚠ |  | ✓ |  |  |
| `mail-reply-all` | ⚠ | ⚠ |  | ✓ |  |  |
| `mail-reply-sender` | ⚠ | ⚠ |  | ✓ |  |  |
| `mail-send` | ⚠ | ⚠ |  | ✓ |  |  |
| `media-eject` | ✗ | ✓ |  | ✓ |  |  |
| `media-playback-start` | ✓ | ⚠ |  | ✓ |  |  |
| `media-seek-backward` | ✓ | ⚠ |  | ✓ |  |  |
| `media-seek-forward` | ✓ | ⚠ |  | ✓ |  |  |
| `process-stop` | ⚠ | ⚠ |  | ✓ |  |  |
| `system-run` | ⚠ | ✓ |  | ✓ |  |  |
| `system-search` | ✗ | ✓ |  | ✗ |  |  |
| `view-fullscreen` | ⚠ | ✓ |  | ✓ |  |  |
| `view-refresh` | ⚠ | ✓ |  | ✓ |  |  |
| `view-restore` | ⚠ | ✓ |  | ✓ |  |  |
| `view-sort-ascending` | ✓ | ✓ |  | ✗ |  |  |
| `view-sort-descending` | ✓ | ✓ |  | ✗ |  |  |
| `window-new` | ⚠ | ✗ |  | ✗ |  |  |
| `zoom-in` | ⚠ | ✓ |  | ✓ |  |  |
| `zoom-original` | ⚠ | ✓ |  | ✗ |  |  |
| `zoom-out` | ⚠ | ✓ |  | ✓ |  |  |

### animations (target sizes: 16 22 32)

**Exist, but with size gaps or wrong pixel size (1):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `process-working` | ✗ | ✓ |  | ✓ |  |  |

### apps (target sizes: 16 22 32 48 64)

**Missing entirely (19):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `accessories-calculator` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kcalc (16,32,48) |
| `accessories-character-map` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kcharselect (16,48) |
| `accessories-dictionary` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kdict (16,32,48) |
| `accessories-screenshot-tool` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/ksnapshot (16,32,48) |
| `accessories-text-editor` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kedit or apps/kwrite (16,32,48) |
| `help-browser` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/help_index (16,32,48) |
| `multimedia-volume-control` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kmix (16,32,48) |
| `preferences-desktop-accessibility` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/access (16,32,48) |
| `preferences-desktop-font` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/fonts (16,32,48) |
| `preferences-desktop-keyboard` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/keyboard (16,32,48) |
| `preferences-desktop-locale` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/locale (16,32,48) |
| `preferences-desktop-multimedia` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/multimedia (16,32,48) |
| `preferences-desktop-screensaver` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kscreensaver (16,32,48) |
| `preferences-desktop-theme` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/looknfeel or apps/kthememgr (16,32,48) |
| `preferences-desktop-wallpaper` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/background (16,32) |
| `system-software-install` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/package_applications (16,32,48) |
| `system-software-update` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kpackage (16,32,48) + arrow overlay |
| `utilities-system-monitor` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/ksysguard (16,32,48) |
| `utilities-terminal` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/konsole or apps/terminal (16,32,48) |

**Exist, but with size gaps or wrong pixel size (1):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `system-file-manager` | ✓ | ✗ |  | ✓ | ✓ | ✗ |

### categories (target sizes: 16 22 32 48)

**Missing entirely (5):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `applications-accessories` | ✗ | ✗ |  | ✗ | ✗ |  | categories/applications-utilities or applications-editors (16,32,48) |
| `applications-engineering` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing (or adapt apps/kivio) |
| `applications-science` | ✗ | ✗ |  | ✗ | ✗ |  | apps/edu_science (16,32) |
| `preferences-other` | ✗ | ✗ |  | ✗ | ✗ |  | categories/applications-settings (16,32,48) |
| `preferences-system-network` | ✗ | ✗ |  | ✗ | ✗ |  | filesystems/network (16,32,48) |

**Exist, but with size gaps or wrong pixel size (14):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `applications-development` | ⚠ | ✗ |  | ✗ | ✗ |  |
| `applications-games` | ✓ | ✗ |  | ✓ | ✓ |  |
| `applications-graphics` | ⚠ | ✗ |  | ⚠ | ⚠ |  |
| `applications-internet` | ⚠ | ✗ |  | ⚠ | ⚠ |  |
| `applications-multimedia` | ⚠ | ✗ |  | ⚠ | ✓ |  |
| `applications-office` | ⚠ | ✗ |  | ⚠ | ✓ |  |
| `applications-other` | ✗ | ✗ |  | ✓ | ✓ |  |
| `applications-system` | ⚠ | ✗ |  | ✓ | ✓ |  |
| `applications-utilities` | ⚠ | ✗ |  | ⚠ | ✓ |  |
| `preferences-desktop` | ⚠ | ✗ |  | ⚠ | ✓ |  |
| `preferences-desktop-peripherals` | ✓ | ✗ |  | ✓ | ✓ |  |
| `preferences-desktop-personal` | ✓ | ✗ |  | ✓ | ✓ |  |
| `preferences-system` | ✓ | ✗ |  | ✓ | ✗ |  |
| `system-help` | ⚠ | ✗ |  | ⚠ | ✓ |  |

### devices (target sizes: 16 22 32 48 64)

**Missing entirely (19):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `audio-card` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kcmsound (16,32) |
| `audio-input-microphone` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `battery` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/laptop_battery (16,32,48) |
| `camera-video` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/xawtv (32) or new |
| `camera-web` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `computer` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kdeprint_computer (16); larger sizes new |
| `drive-optical` | ✗ | ✗ |  | ✗ | ✗ | ✗ | derive from devices/media-optical-cd (16,32,48) |
| `drive-removable-media` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/usb (16,32,48) or devices/media-zip (16,32,48) |
| `input-gaming` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `media-flash` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `media-floppy` | ✗ | ✗ |  | ✗ | ✗ | ✗ | devices/media-floppy-350 (16,32,48) — copy |
| `media-optical` | ✗ | ✗ |  | ✗ | ✗ | ✗ | devices/media-optical-cd (16,32,48) — copy |
| `media-tape` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kdat (16,32,48) |
| `modem` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kppp (16,32,48) |
| `multimedia-player` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing (or adapt apps/noatun) |
| `network-wired` | ✗ | ✗ |  | ✗ | ✗ | ✗ | filesystems/network (16,32,48) |
| `network-wireless` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `phone` | ✗ | ✗ |  | ✗ | ✗ | ✗ | new drawing |
| `video-display` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kcmx (16,32) |

**Exist, but with size gaps or wrong pixel size (8):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `camera-photo` | ⚠ | ✗ |  | ✓ | ✗ | ✗ |
| `drive-harddisk` | ⚠ | ✗ |  | ✓ | ✓ | ✗ |
| `input-keyboard` | ✓ | ✗ |  | ✓ | ✓ | ✗ |
| `input-mouse` | ✓ | ✗ |  | ⚠ | ✓ | ✗ |
| `input-tablet` | ⚠ | ✗ |  | ✓ | ✓ | ✗ |
| `pda` | ✗ | ✗ |  | ⚠ | ✗ | ✗ |
| `printer` | ⚠ | ✗ |  | ⚠ | ✓ | ✗ |
| `scanner` | ⚠ | ✗ |  | ⚠ | ✓ | ✗ |

### emblems (target sizes: 16 24 32)

**Missing entirely (11):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `emblem-default` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-documents` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-downloads` | ✗ |  | ✗ | ✗ |  |  | new drawing (arrow, cf. apps/kget 48) |
| `emblem-favorite` | ✗ |  | ✗ | ✗ |  |  | scale down categories/applications-favorite |
| `emblem-important` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-mail` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-photos` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-readonly` | ✗ |  | ✗ | ✗ |  |  | filesystems/lockoverlay (16,32,48) |
| `emblem-shared` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-synchronized` | ✗ |  | ✗ | ✗ |  |  | new drawing |
| `emblem-system` | ✗ |  | ✗ | ✗ |  |  | new drawing |

**Exist, but with size gaps or wrong pixel size (1):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `emblem-unreadable` | ✓ |  | ✓ | ✗ |  |  |

### mimetypes (target sizes: 16 22 32 48 64)

**Missing entirely (8):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `application-x-executable` | ✗ | ✗ |  | ✗ | ✗ | ✗ | mimetypes/binary (16,32,48) |
| `font-x-generic` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/fonts (16,32,48) on a document page |
| `package-x-generic` | ✗ | ✗ |  | ✗ | ✗ | ✗ | mimetypes/tgz or tar (16,32,48,64) |
| `text-x-generic-template` | ✗ | ✗ |  | ✗ | ✗ | ✗ | derive from text-x-generic (16,32,48,64) |
| `text-x-script` | ✗ | ✗ |  | ✗ | ✗ | ✗ | mimetypes/shellscript (16,32,48,64) |
| `x-office-address-book` | ✗ | ✗ |  | ✗ | ✗ | ✗ | mimetypes/vcard (16,32,48) |
| `x-office-calendar` | ✗ | ✗ |  | ✗ | ✗ | ✗ | mimetypes/vcalendar (16,32,48) |
| `x-office-presentation` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kpresenter (16,32,48) on a document page |

**Exist, but with size gaps or wrong pixel size (7):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `audio-x-generic` | ⚠ | ✗ |  | ⚠ | ✓ | ✓ |
| `image-x-generic` | ⚠ | ✗ |  | ⚠ | ✓ | ✓ |
| `text-html` | ⚠ | ✗ |  | ⚠ | ⚠ | ✓ |
| `text-x-generic` | ⚠ | ✗ |  | ⚠ | ✓ | ✓ |
| `video-x-generic` | ⚠ | ✗ |  | ⚠ | ✓ | ✓ |
| `x-office-document` | ⚠ | ✗ |  | ⚠ | ✓ | ✓ |
| `x-office-spreadsheet` | ✗ | ✗ |  | ⚠ | ✓ | ✗ |

### places (target sizes: 16 22 32 48 64)

**Missing entirely (5):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `folder-remote` | ✗ | ✗ |  | ✗ | ✗ | ✗ | filesystems/ftp (16,32,48) |
| `network-server` | ✗ | ✗ |  | ✗ | ✗ | ✗ | filesystems/services (16,32,48) or new |
| `network-workgroup` | ✗ | ✗ |  | ✗ | ✗ | ✗ | filesystems/network or apps/samba (16,32,48) |
| `start-here` | ✗ | ✗ |  | ✗ | ✗ | ✗ | apps/kmenu (16,32,48) |
| `user-bookmarks` | ✗ | ✗ |  | ✗ | ✗ | ✗ | actions/bookmark (16,22,32) / apps/keditbookmarks (16,32) |

**Exist, but with size gaps or wrong pixel size (4):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 |
|---|---|---|---|---|---|---|
| `folder` | ✓ | ⚠ |  | ⚠ | ✓ | ✓ |
| `user-desktop` | ✗ | ✗ |  | ⚠ | ✓ | ✗ |
| `user-home` | ⚠ | ✗ |  | ⚠ | ⚠ | ✗ |
| `user-trash` | ⚠ | ✗ |  | ⚠ | ✓ | ✗ |

### status (target sizes: 16 22 32 48)

**Missing entirely (59):**

| Icon | 16 | 22 | 24 | 32 | 48 | 64 | Suggested source (sizes available) |
|---|---|---|---|---|---|---|---|
| `appointment-missed` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/bell / apps/kalarm |
| `appointment-soon` | ✗ | ✗ |  | ✗ | ✗ |  | actions/kalarm (16,22) / apps/kalarm (32,48) |
| `audio-volume-high` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/volume (16,22,32) |
| `audio-volume-low` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/volume |
| `audio-volume-medium` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/volume |
| `audio-volume-muted` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/volume + strike-through |
| `battery-caution` | ✗ | ✗ |  | ✗ | ✗ |  | derive from apps/laptop_battery |
| `battery-low` | ✗ | ✗ |  | ✗ | ✗ |  | derive from apps/laptop_battery |
| `dialog-error` | ✗ | ✗ |  | ✗ | ✗ |  | apps/error (32); other sizes new |
| `dialog-information` | ✗ | ✗ |  | ✗ | ✗ |  | mimetypes/info (16,32,48) |
| `dialog-password` | ✗ | ✗ |  | ✗ | ✗ |  | apps/password (16,32,48) |
| `dialog-question` | ✗ | ✗ |  | ✗ | ✗ |  | actions/contexthelp or help (16,22,32) |
| `dialog-warning` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing (yellow triangle) |
| `folder-drag-accept` | ✗ | ✗ |  | ✗ | ✗ |  | derive from places/folder |
| `folder-open` | ✗ | ✗ |  | ✗ | ✗ |  | derive from places/folder |
| `folder-visiting` | ✗ | ✗ |  | ✗ | ✗ |  | derive from places/folder |
| `image-loading` | ✗ | ✗ |  | ✗ | ✗ |  | new (or a frame of animations/process-working) |
| `image-missing` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `mail-attachment` | ✗ | ✗ |  | ✗ | ✗ |  | actions/attach (16,22,32) |
| `mail-unread` | ✗ | ✗ |  | ✗ | ✗ |  | derive from mimetypes/message |
| `mail-read` | ✗ | ✗ |  | ✗ | ✗ |  | derive from mimetypes/message |
| `mail-replied` | ✗ | ✗ |  | ✗ | ✗ |  | derive from mimetypes/message + mail-reply-sender arrow |
| `mail-signed` | ✗ | ✗ |  | ✗ | ✗ |  | actions/signature (16,22,32) |
| `mail-signed-verified` | ✗ | ✗ |  | ✗ | ✗ |  | actions/signature + check mark |
| `media-playlist-repeat` | ✗ | ✗ |  | ✗ | ✗ |  | actions/noatunloopplaylist (22) |
| `media-playlist-shuffle` | ✗ | ✗ |  | ✗ | ✗ |  | actions/noatunshuffle (22) |
| `network-error` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `network-idle` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `network-offline` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `network-receive` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `network-transmit` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `network-transmit-receive` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `printer-error` | ✗ | ✗ |  | ✗ | ✗ |  | apps/kdeprint_printer_defect (16) |
| `printer-printing` | ✗ | ✗ |  | ✗ | ✗ |  | apps/kdeprint_printer_process (16,32) |
| `security-high` | ✗ | ✗ |  | ✗ | ✗ |  | actions/encrypted (16,22,32) |
| `security-medium` | ✗ | ✗ |  | ✗ | ✗ |  | actions/halfencrypted (16,22,32) |
| `security-low` | ✗ | ✗ |  | ✗ | ✗ |  | actions/decrypted (16,22,32) |
| `software-update-available` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `software-update-urgent` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `sync-error` | ✗ | ✗ |  | ✗ | ✗ |  | derive from sync-synchronizing |
| `sync-synchronizing` | ✗ | ✗ |  | ✗ | ✗ |  | actions/busysync (16) |
| `task-due` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/todo (16,22,32) |
| `task-past-due` | ✗ | ✗ |  | ✗ | ✗ |  | derive from actions/todo (16,22,32) |
| `user-available` | ✗ | ✗ |  | ✗ | ✗ |  | derive from apps/licq (16,32,48) or new |
| `user-away` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `user-idle` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `user-offline` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `user-trash-full` | ✗ | ✗ |  | ✗ | ✗ |  | mimetypes/recycled (16,32,48,64) or derive from user-trash |
| `weather-clear` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-clear-night` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-few-clouds` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-few-clouds-night` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-fog` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-overcast` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-severe-alert` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-showers` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-showers-scattered` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-snow` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |
| `weather-storm` | ✗ | ✗ |  | ✗ | ✗ |  | new drawing |

