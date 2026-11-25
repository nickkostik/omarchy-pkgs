# Keep handwritten PDF signatures visible after saving in Papers

Review notes for the published draft and its synthetic samples. [Omarchy Packages PR #3395](https://github.com/omacom/omarchy-pkgs/pull/3395)

## What happened

I drew a signature in Omarchy’s Document Viewer, saved the PDF, and emailed it. The recipients couldn’t see the signature. I could see and print it from the same attachment in Gmail on my iPhone. Printing from my desktop could also leave it off the page. The screenshot workaround made the signature visible but cost me selectable text and interactive fields.

## Why the signature disappears

In Papers 51.1 with Poppler 26.10, the Pencil strokes I tested were saved as PDF `/Ink` annotations without a Print flag or a stored appearance stream. The Print flag helps viewers that include annotations when printing. If a viewer renders only the page, the signature needs to be in the page contents. [Papers issue](https://gitlab.gnome.org/GNOME/papers/-/work_items/645) · [Upstream Print flag fix](https://gitlab.gnome.org/GNOME/papers/-/merge_requests/823)

## What this changes

- Backport the upstream Print flag fix as its own patch. It also helps when printing from Papers before saving and when a PDF keeps its ink as annotations.
- In the second patch, **Save As** embeds eligible Pencil strokes from the current editing session as vector paths. It preserves their position, width, color, and opacity and removes the corresponding annotation so each stroke draws once. Existing review ink stays editable with its original print setting. Links, form fields, and other annotations stay in the PDF.
- The save dialog describes when Pencil ink may become permanent. **Save Editable Copy** keeps the Pencil strokes editable in the saved copy. Undo and redo still work in the open editing session.
- Saves with no new eligible strokes skip the qpdf rewrite. Encrypted and certificate-signed PDFs use Papers’ ordinary save path. Local saves write a complete copy before replacing the destination and keep edits available if the write fails.

The package uses qpdf to edit PDF objects and rebuilds when qpdf changes. A Papers version bump needs a review of both patches. Form widgets still render in the annotation layer, so an opaque widget can cover page ink where they overlap.

## What I verified

- Built Papers 51.1-1.2 with `makepkg --check` using a cached local x86_64 build directory. All 10 Meson test groups passed, including four ink save cases. The tests cover scoped ink, rotated and cropped pages, stroke width and opacity, form field editing, repeated saves, undo and redo, cancellation, failed writes, retry, and encryption. Stock Papers 51.1 fails the original portability regression.
- Drew a synthetic signature in the Papers UI and saved the same drawing both ways. Chromium and Ghostscript rendered the embedded copy. Papers’ **Print to File** included the signature once, and the original page text was still selectable. Both examples are synthetic. [Editable annotation copy](https://github.com/nickkostik/omarchy-pkgs/blob/cd86a014f0cf652a9db792afa7279fd4aaf9031c/papers-ink-evidence/annotation-copy.pdf) · [Embedded page content copy](https://github.com/nickkostik/omarchy-pkgs/blob/cd86a014f0cf652a9db792afa7279fd4aaf9031c/papers-ink-evidence/page-content-copy.pdf)
- Confirmed that a certificate-signed sample takes the same path as an editable save. The certificate signature over the earlier revision still validates after the ink is added as an annotation.
- Native x86_64 and aarch64 CI are awaiting maintainer approval. The build run for this revision has no jobs yet. [Build run](https://github.com/omacom/omarchy-pkgs/actions/runs/45570798462)

To compare the two PDFs, run `pdftoppm -hide-annotations -png -singlefile` on each. Only the embedded copy shows the signature when annotations are hidden.

## If carrying the second patch is out of scope

The upstream Print flag fix can ship on its own. For sharing, I’d propose **Save Flattened Copy** upstream in Papers so users can choose when to embed handwritten strokes in a portable copy. A stored Pencil appearance in Poppler would also help viewers that support annotations but need the appearance stream.
