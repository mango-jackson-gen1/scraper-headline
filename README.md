# Baidu Hot Search Tracker

This project scrapes the top headlines from Baidu's real-time hot search board and saves them to a CSV every four hours. Over time you get a running record of what was trending on China's largest search engine, timestamped so you can look back at any point in the day.

The live board is here: [top.baidu.com/board?tab=realtime](https://top.baidu.com/board?tab=realtime).

## What gets collected

Each run pulls the top 50 or so items and writes one row per headline to `baidu-scrape1.csv`. The columns:

| Column | What it is |
| --- | --- |
| `rank` | Position on the board (the featured top item has no rank) |
| `title` | The headline |
| `subhead` | The short description underneath |
| `count` | Baidu's "hot index" score for that item |
| `tag` | Label like 热 (hot) or 新 (new), if any |
| `link` | Baidu search URL for the topic |
| `scrape_date` | When this row was collected, e.g. `2026-07-03_12.35` |

New scrapes get appended to the top of the file, so the CSV just keeps growing. It's currently a few thousand rows and counting.

## How it works

There are three moving parts.

**The scraper** — `test1-headline.ipynb`. A short notebook that requests the board page, parses it with BeautifulSoup, pulls the fields above out of each headline block, and appends the results to the CSV with the current timestamp. The parsing leans on Baidu's CSS class names (`category-wrap_iQLoo`, `hot-index_1Bl1a`, and so on), which is the fragile part — see the notes below.

**The automation** — `.github/workflows/main.yml`. A GitHub Actions workflow that runs the notebook on a schedule (`cron: 0 */4 * * *`, so every 4 hours) and also whenever you trigger it by hand from the Actions tab. After each run it commits the updated CSV back to the repo. That's why the commit history is a long list of "Latest data: ..." messages.

**The display** — `index.html`. A single page that embeds a [Datawrapper](https://www.datawrapper.de/) table showing the current data. Datawrapper reads from the CSV and renders the table; the HTML is just the embed.

## Running it yourself

You don't need to run anything locally for it to keep collecting — that happens on GitHub. But if you want to run the scraper on your own machine:

```bash
pip install jupyter pandas requests beautifulsoup4 lxml html5lib
jupyter nbconvert --to notebook --execute test1-headline.ipynb
```

That executes the notebook end to end and updates `baidu-scrape1.csv` in place. Or just open the notebook and run the cells top to bottom.

## Notes and caveats

- The scraper depends on Baidu's page structure and its hashed CSS class names. When Baidu changes those (and they do), the scrape will start returning blanks or break, and the selectors in the notebook need updating.
- The count is Baidu's own hot-index number, not a raw view or search count. Treat it as a relative ranking signal, not an absolute figure.
- Everything is scraped as-is in Chinese. There's no translation step.
- Row count in the top item: the single "featured" headline at the top of the board doesn't carry a numeric rank, so its `rank` field is empty.

## Where this came from

Built as a data-journalism scraping exercise, following [Jonathan Soma's scraper sample](https://github.com/jsoma/scraper-sample) and [this walkthrough](https://www.youtube.com/watch?v=QNKxzkNpsko) for the scheduled-scraping setup.
