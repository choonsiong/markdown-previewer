# Markdown previewer

A single-page markdown previewer built with plain HTML, CSS and JavaScript. No build step, no server.

- Paste or type markdown, or upload / drag in a `.md` file
- Live GitHub-flavoured preview with syntax-highlighted code, tables and task lists
- Export the rendered document as an A4 PDF
- Follows your system light/dark setting; works on phones

## Run it

Open `index.html` in a browser, or host the folder anywhere static files are served (GitHub Pages, Netlify, an S3 bucket…).

## Deploy to GitHub Pages

1. Push this folder to a repository (`index.html` at the root).
2. In the repository go to **Settings → Pages**.
3. Under **Build and deployment**, set *Source* to **Deploy from a branch**, pick your branch (usually `main`) and the **/ (root)** folder, then save.
4. After a minute the site is served at `https://<your-username>.github.io/<repository-name>/`.

## Libraries (loaded from cdnjs)

- [marked](https://github.com/markedjs/marked) 12.0.2 – markdown parsing
- [DOMPurify](https://github.com/cure53/DOMPurify) 3.0.8 – HTML sanitising
- [highlight.js](https://highlightjs.org/) 11.9.0 – code highlighting
- [html2pdf.js](https://github.com/eKoopmans/html2pdf.js) 0.10.1 – PDF export
- IBM Plex fonts via Google Fonts
