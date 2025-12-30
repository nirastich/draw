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
- The drawing is stored in the URL **hash** (`#...`), not on a server.
- Each stroke is recorded as:
  - color (RGB)
  - thickness
  - a list of points
- Points are stored as normalized coordinates (0..1) and quantized to 16-bit integers. This keeps the data compact and resolution-independent.
- The full stroke list is serialized into a small binary format and then encoded into the hash.
- To keep URLs short, it chooses the smallest encoding among a few options (raw base64url, LZ-based compression, and optionally browser compression streams).
- On load (or when the hash changes), the page decodes the hash, rebuilds the stroke list, and redraws the canvas.
- The PNG download button exports the current canvas with `toBlob` (or `toDataURL` as a fallback).

## Limits
- Because everything is in the URL hash, very complex drawings can hit browser and platform URL limits.
- Exported PNG size depends on the current viewport resolution.
