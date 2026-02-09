# DRAW (draw.73.nu)

A minimal drawing canvas that stores strokes in the URL.

**Live:** https://draw.73.nu

## What it does

- Draw freehand strokes on a full-screen canvas
- Pick color and thickness
- Undo strokes
- Fit the drawing to the current viewport aspect (without scaling existing strokes)
- Share the drawing by sharing the URL
- Download the current canvas as a PNG

## How it works

- The drawing is stored in the URL **hash** (`#…`), not on a server.
- Each stroke is recorded as:
  - color (RGB)
  - thickness
  - a list of points
- Points are stored as normalized coordinates (0..1) and quantized to 16-bit integers. This keeps the data compact and resolution-independent.
- The full stroke list is serialized into a compact binary format (V3) and then encoded into the hash.
- V3 cuts binary size by ~43-45% over V2 (before compression) through:
  - **Delta-encoded points:** only the first point per stroke is absolute (2×uint16). The rest are stored as deltas from the previous point using zigzag varints, so small movements take 1-2 bytes per axis instead of a fixed 4.
  - **Stroke deduplication:** a flags byte marks whether color/thickness match the previous stroke. When they do, those bytes are skipped.
  - **Varint counts:** stroke and point counts use variable-length integers instead of fixed uint16.
- Old V2 URLs still load fine; the parser checks the version byte in the header.
- To keep URLs short, it chooses the smallest encoding among a few options (raw base64url, LZ-based compression, and optionally browser compression streams). The delta-encoded data also compresses better than V2 did, so real-world URL savings tend to be larger than the raw binary difference.
- On load (or when the hash changes), the page decodes the hash, rebuilds the stroke list, and redraws the canvas.
- The PNG download button exports the current canvas with `toBlob` (or `toDataURL` as a fallback).

## Limits

- Because everything is in the URL hash, very complex drawings can hit browser and platform URL limits.
- Exported PNG size depends on the current viewport resolution.
