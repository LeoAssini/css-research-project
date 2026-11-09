# Validating the ordeal axis on Guardian news

Can a single direction in a word embedding tell apart news about *ordeals* from news about *natural dangers*?

Following Morin et al. (2019), an **ordeal** is a rare, severe threat where a person's reaction can dramatically change the outcome, such as murder, war or assault. A **natural danger** is a threat more or less beyond our control, such as an earthquake, flood or epidemic.

This project builds an "ordeal axis" from pretrained Google News word2vec vectors, using the grouped-pairs method of Kozlowski et al. (2019): the average of 10 ordeal anchor words minus the average of 10 natural danger anchor words. It then scores 1,500 Guardian articles on that axis, 500 each from ordeal, natural danger and control (unrelated topics) tags, and tests whether ordeal articles score higher than natural danger articles.

Everything is in one notebook, [01_embedding_google_news.ipynb](01_embedding_google_news.ipynb).

## How to run the notebook

### Requirements

- Python 3.9 (the notebook was last run on 3.9.6)
- About 6 GB of free disk space: 1.6 GB for the raw embedding download in `~/gensim-data`, and 3.7 GB for the normalized copy in `data/`
- A free [Guardian Open Platform developer key](https://open-platform.theguardian.com/access/), because the article text is not included in this repository

### Steps

1. From the project folder, create an environment with the package versions used for the saved results:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install gensim==4.4.0 numpy==2.0.2 pandas==2.3.3 scipy==1.13.1 matplotlib==3.9.4 requests==2.32.5 ipykernel
   ```

2. Open the notebook in VS Code or Jupyter, select the `.venv` kernel, and make sure the working directory is the project folder, because the data paths are relative to it.

3. Put your key in the `GUARDIAN_API_KEY` environment variable, then restart your editor or Jupyter so the notebook can see it. On macOS with zsh:

   ```bash
   echo 'export GUARDIAN_API_KEY="your-key-here"' >> ~/.zshrc
   ```

   The key is only needed for API calls that are not already in the cache.

4. Run all cells from top to bottom.

### What to expect

- **Articles:** by default the notebook downloads exactly the articles listed in `data/guardian/article_index.csv`, using about 40 API calls. It reports any article that has since been removed or edited, and never swaps in a different one.
- **Embedding:** the first run downloads and normalizes the vectors, which holds about 3.6 GB in memory. Later runs load the saved copy in seconds.
- **Hand ratings:** section 1.5 already contains the ratings for the 90 hand-checked articles. To redo them blind, read `data/guardian/hand_check/articles_to_read.md`, enter your scores in `RATINGS` before opening `sample_key.csv`, and then run the cell.
- **`matmul` warnings:** the `RuntimeWarning`s from `@` in sections 3.1, 3.2 and 3.5 are harmless. Recomputing the article scores another way gives the same values.

### Choosing a different sample

New articles are only chosen if you change something. Changing `SEARCH_TERMS` (section 1.1), the tag rules such as `EXCLUDE_TAGS` (section 1.2), or a sample setting such as `ARTICLES_PER_GROUP` or `SEED` (section 1.3) makes the notebook run the selection again and overwrite the saved sample.

A new selection makes about 200 to 300 API calls, plus one for each new tag, and a developer key allows 500 calls per day. If the limit is hit, rerun the failed cell the next day. Every response is cached in `data/guardian/cache/`, so finished calls are not repeated. The hand ratings in section 1.5 belong to the original sample and will need redoing.

## Reproducibility

Random steps use fixed seeds, and the chosen tags and articles are saved, so a rerun uses the same sample however much the Guardian publishes later. The article text belongs to the Guardian and is not in this repository, so the files marked below are recreated when you run the notebook.

| File | Contents |
| :- | :- |
| `data/guardian/group_tags.json` | The tags chosen for each group |
| `data/guardian/sample_settings.json` | The settings used for the download |
| `data/guardian/tag_catalogue.csv`, `tag_candidates.csv` | The tags considered and their article counts |
| `data/guardian/article_index.csv` | The chosen articles, in order, with a hash of each text |
| `data/guardian/articles.jsonl` | The articles with their full text (recreated) |
| `data/guardian/hand_check/` | The hand-check sample and ratings (`articles_to_read.md` is recreated) |
| `data/word_lists.json` | The stopwords and anchor word forms |
| `data/guardian/tokens.jsonl` | The tokens kept for each article (recreated) |
| `data/ordeal_axis.npy` | The ordeal axis |
| `data/guardian/scores.csv` | The score of each article |
| `data/guardian/axis_test.json` | The test results and verdict |
