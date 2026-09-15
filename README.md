# Researcher / Resume Website

A lightweight personal academic website built with plain HTML, CSS, and JavaScript
(no build tools or frameworks required). It includes sections for a bio, research
interests, publications, experience, education, skills, and contact info, plus a
link to a downloadable resume/CV PDF.

## Structure

```
index.html       Main page and content
css/style.css     Styling (supports light/dark mode automatically)
js/main.js        Mobile nav toggle + footer year
assets/           Put your profile photo (profile.jpg) and resume.pdf here
```

## Customize

Open `index.html` and replace the placeholder text:

- Name, title, affiliation, and tagline in the `.hero` section
- Social links (email, Google Scholar, GitHub, LinkedIn, ORCID)
- Research interests tags
- Publications list (title, authors, venue, PDF/code/BibTeX links)
- Experience and education timelines
- Skills tags
- Contact details

Add your own images:

- `assets/profile.jpg` — your photo (shows a placeholder if missing)
- `assets/resume.pdf` — your CV, linked from the "Resume (PDF)" button in the nav

## Run locally

No build step needed. Just open `index.html` in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy for free (GitHub Pages)

1. Push this repo to GitHub (already done if you're reading this from the repo).
2. In the repo settings, go to **Pages**.
3. Under "Build and deployment", set **Source** to "Deploy from a branch".
4. Pick the branch (e.g. `main`) and root folder `/`, then save.
5. Your site will be live at `https://<username>.github.io/<repo-name>/`.

Other free options: Netlify, Vercel, or Cloudflare Pages — all support dragging
this folder in or connecting the GitHub repo directly, with no config needed
since this is static HTML/CSS/JS.
