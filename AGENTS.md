# Personal homepage

This repository owns Jerry Lingjie Mei's personal homepage.

## Publishing

- GitHub Pages publishes `main` from the repository root at https://jerrylingjiemei.github.io/.
- The homepage is also published on the Princeton CS website at https://www.cs.princeton.edu/~lm5483/.
- `origin` and `pages` point to the same GitHub repository. One push updates both remote destinations.
- The Princeton CS copy is an existing Git checkout at `/n/fs/grad/lm5483/public_html`, accessible through `ssh wash`. After pushing, run `git -C /n/fs/grad/lm5483/public_html pull --ff-only origin main` there. Preserve unrelated files, including `gpu-dashboard/`.
- Verify both public sites after publishing. A successful GitHub Pages deployment does not prove the Princeton CS copy is current.

## Content and ownership

- `index.html` owns the biography, research statement, writing links, publications, and project summaries.
- `css/` and `js/` own styling and publication filtering.
- `cv/` owns the CV source and PDF. `projects/` owns individual research project pages.
- Keep homepage edits scoped. Preserve publication links and distinguish research interests from completed results.
