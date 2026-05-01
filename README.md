# Researcher Personal Dashboard

A self-hosted personal dashboard that auto-fetches arXiv preprints (with keyword highlighting), physics news (APS, Nature), and citation metrics from Semantic Scholar — updated 4× per day via GitHub Actions, served for free on GitHub Pages.

## Live demo structure

```
researcher-dashboard/
├── index.html              ← the dashboard (your browser opens this)
├── data/
│   ├── arxiv.json          ← written by GitHub Actions
│   ├── scholar.json        ← written by GitHub Actions
│   ├── news.json           ← written by GitHub Actions
│   └── meta.json           ← timestamp + config snapshot
├── scripts/
│   └── fetch_data.py       ← the Python fetcher
└── .github/workflows/
    └── fetch.yml           ← GitHub Actions schedule
```

---

## Setup (10 minutes)

### 1. Create the GitHub repository

```bash
cd researcher-dashboard
git init
git add .
git commit -m "initial commit"
```

Go to [github.com/new](https://github.com/new), create a **public** repo named `researcher-dashboard` (must be public for free GitHub Pages), then:

```bash
git remote add origin https://github.com/YOUR_USERNAME/researcher-dashboard.git
git branch -M main
git push -u origin main
```

### 2. Enable GitHub Pages

- Go to your repo → **Settings** → **Pages**
- Under **Source**, select: `Deploy from a branch`
- Branch: `main`, folder: `/ (root)`
- Click **Save**

Your dashboard will be live at:
```
https://YOUR_USERNAME.github.io/researcher-dashboard/
```

Bookmark that URL — this is your personal dashboard.

### 3. Set your arXiv preferences (GitHub Variables)

Go to **Settings** → **Secrets and variables** → **Actions** → **Variables** tab → **New repository variable**:

| Variable name       | Example value                              |
|---------------------|--------------------------------------------|
| `ARXIV_CATEGORIES`  | `cond-mat,quant-ph`                        |
| `ARXIV_KEYWORDS`    | `topological,ARPES,nodal-line,semimetal`   |

Multiple categories/keywords are comma-separated, no spaces.

### 4. Set your Semantic Scholar author ID (Secret)

1. Go to [semanticscholar.org](https://www.semanticscholar.org), search your name
2. Open your author profile
3. Copy the **numeric ID** from the URL:  
   `semanticscholar.org/author/Iftakhar-Bin-Elius/` **`123456789`**

Go to **Settings** → **Secrets and variables** → **Actions** → **Secrets** tab → **New repository secret**:

| Secret name            | Value               |
|------------------------|---------------------|
| `SEMANTIC_SCHOLAR_ID`  | `123456789`         |

### 5. Run the Action manually (first time)

- Go to your repo → **Actions** → **Fetch Research Data** → **Run workflow**
- Wait ~30 seconds, then reload your dashboard

After this, data refreshes automatically at **00:00, 06:00, 12:00, 18:00 UTC** every day.

---

## Customization

### Add more arXiv categories

Edit the `ARXIV_CATEGORIES` variable:
```
cond-mat,quant-ph,physics.comp-ph,hep-th
```

### Add more news feeds

Edit `scripts/fetch_data.py`, find the `NEWS_FEEDS` list, and add entries:
```python
("Science",  "https://www.science.org/rss/news_current.xml"),
("PRL",      "https://feeds.aps.org/rss/recent/prl.xml"),
```

### Change fetch frequency

Edit `.github/workflows/fetch.yml`, the `cron` line. Example for every 3 hours:
```yaml
- cron: "0 */3 * * *"
```

---

## Notes on Google Scholar

Google Scholar has no public API and actively blocks automated access. Semantic Scholar is the correct alternative for physics research — it covers all arXiv preprints and major journal publications with no authentication required. Coverage for condensed matter and quantum materials is excellent.

If you specifically need Google Scholar metrics (e.g., for an exact citation count that differs), the `scholarly` Python library can scrape it, but it requires either:
- A proxy/VPN rotation setup, or
- Running on a university network

Add it to `fetch_data.py` if needed:
```python
from scholarly import scholarly
author = next(scholarly.search_author('Iftakhar Bin Elius'))
scholarly.fill(author)
```

---

## Troubleshooting

**Dashboard shows "not yet fetched":** trigger the Action manually (step 5 above).

**Scholar shows error:** verify the numeric ID is correct; test it:
```
https://api.semanticscholar.org/graph/v1/author/YOUR_ID?fields=name
```

**arXiv shows no keyword highlights:** check `ARXIV_KEYWORDS` variable spelling; the match is case-insensitive substring match against title + abstract.
