# Assets

Brand/icon source assets for CopyCat.

## `icons/`

The "paws" mark, supplied in several sizes plus a vector source:

| File | Use |
|------|-----|
| `paws.svg` | Vector master |
| `paws_512.png` … `paws_32.png` | Raster exports (512/256/128/64/32) |

These source images are the single source of truth for:

- The **GUI** icon set under [`gui/src-tauri/icons/`](../gui/src-tauri/icons/)
  (`icon.ico`, `icon.icns`, `icon.png`, `32x32.png`, `128x128.png`,
  `128x128@2x.png`), referenced by [`tauri.conf.json`](../gui/src-tauri/tauri.conf.json).
- The GUI window header logo ([`gui/ui/logo.png`](../gui/ui/logo.png)).
- The project README logo.

To regenerate the GUI icon set after changing the source images, rebuild the
`icon.ico`/`icon.icns` (multi-size, PNG-based) and copy the sized PNGs into
`gui/src-tauri/icons/`.
