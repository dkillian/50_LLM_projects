# Project Memory — 50 ML Projects to Understand LLMs (R Translation)

## Overview

This repo is an R/Quarto translation of Mike X Cohen's book "50 ML projects to understand LLMs: Investigate transformer mechanisms through data analysis, visualization, and experimentation." The original Python notebooks are in each chapter folder alongside the R translations.

- Original repo: https://github.com/mikexcohen/ML4LLM_book
- Published output: https://dkillian.github.io/50_LLM_projects/

## File Structure

```
chapter_2/
  R/                          # Quarto translations
    ml4llm_ch2_proj1.qmd
    ml4llm_ch2_proj2.qmd
    ml4llm_ch2_proj3.qmd
    tokenizer.json            # GPT-2 tokenizer saved locally (avoids HuggingFace network call)
    book_emb.rds              # Saved sentence-transformer embeddings for 10 books (proj2 addendum)
  ml4llm_ch2_proj*.ipynb     # Original Python notebooks (helper + solution versions)

misc/R/
  correlation_vs_cosine_sim.qmd   # Supplementary explainer on correlation vs cosine similarity

prep (May 2025).R             # Global setup script (sourced in every qmd)
README.md
```

## Quarto Document Conventions

Every R translation follows this structure:

### YAML front matter
```yaml
---
title: "Descriptive Title"
subtitle: "50 ML Projects to Understand LLMs — Chapter 2, Project N"
author: "Dan Killian, with Posit AI<br>an R translation of Mike X Cohen's original Python notebook"
date: today
format:
  html:
    toc: true
    code-fold: false
    theme: cosmo
    fig-width: 8
    fig-height: 4
execute:
  warning: false
  message: false
---
```

### First chunk (always)
```r
```{r global_options, include=F, warning=F, message=F, echo=F, error=F}
library(here)
library(knitr)
source("../../../prep (May 2025).r")
theme_set(base_quarto)
```
```

### Code style
- Use `|>` base pipe (not `%>%`)
- Use `\(x)` anonymous function syntax
- Use `purrr::map()` for iteration (not for-loops)
- Use `flextable()` for tables with `flextable::set_caption()`
- Use `faceted` theme object (from prep script) on all `facet_wrap()` plots
- Use `scales::label_scientific()` for large numeric axes
- Chunk labels in kebab-case; `fig-cap`, `fig-width`, `fig-height` as chunk options
- Section facet panels labeled "A) ...", "B) ..." via `mutate()` before plotting

### Prose style
- Italic disclaimer at top: `*Using the code without reading the book may lead to confusion or errors.*`
- `---` horizontal rules between major sections
- `#` headers for each part
- Brief prose before each figure (context) and after (interpretation)
- Blockquotes `>` for challenge/task instructions

## Key Objects and Setup

### Books data frame (used in proj2, proj3, and likely beyond)
```r
baseurl <- "https://www.gutenberg.org/cache/epub/"

books <- data.frame(
  code  = c("84", "64317", "11", "1513", "76", "219", "2591", "2148", "36", "829"),
  title = c("Frankenstein", "GreatGatsby", "AliceWonderland", "RomeoJuliet", "HuckFinn",
            "HeartDarkness", "GrimmsTales", "EdgarAllenPoe", "WarOfTheWorlds", "GulliversTravels"),
  label = c("Frankenstein", "The Great Gatsby", "Alice in Wonderland", "Romeo and Juliet",
            "Adventures of Huck Finn", "Heart of Darkness", "Grimm's Tales", "Edgar Allan Poe",
            "War of the Worlds", "Gullivers Travels")
)
```
Use `books$label` for display names in plots and tables; `books$title` for short programmatic names; `books$code` for URL construction.

### GPT-2 tokenizer
Do NOT use `tok::tokenizer$from_pretrained("gpt2")` — this calls HuggingFace at render time and fails on network errors. Load from the saved local file instead:
```r
gpt2 <- tok::tokenizer$from_file("tokenizer.json")
```
`tokenizer.json` is saved at `chapter_2/R/tokenizer.json`. If it needs to be recreated: `gpt2$save("tokenizer.json")`.

### Sentence-transformer embeddings (proj2 addendum)
```r
library(reticulate)
st    <- import("sentence_transformers")
model <- st$SentenceTransformer("all-MiniLM-L6-v2")
```
Book embeddings are cached at `chapter_2/R/book_emb.rds`. If deleted, the semantic search chunk will recompute and re-save them. Do NOT use `cache: true` on chunks containing reticulate Python objects — they cannot be serialized to disk.

### Cosine similarity formula (manual R implementation)
```r
sum(x * y) / (sqrt(sum(x^2)) * sqrt(sum(y^2)))
```

## Prep Script (prep (May 2025).R)

Located at `C:/Users/dkill/OneDrive/Documents/prep (May 2025).R`. Sourced in every qmd. Key objects it provides:
- `base_quarto` — ggplot theme for quarto HTML output
- `base_ppt` — ggplot theme for PowerPoint
- `faceted` — ggplot theme modifier for facet_wrap() plots
- USAID color palette variables (`usaid_blue`, `usaid_red`, etc.)
- Loads tidyverse and many other packages

## Notes

- Project 2 has a semantic search addendum (not in the original Python notebook) that embeds all 10 books using `sentence-transformers` and ranks them by cosine similarity to a query sentence.
- The `misc/R/correlation_vs_cosine_sim.qmd` document explains the mathematical relationship between Pearson correlation and cosine similarity, and includes a simulation showing how mean offsets pull cosine similarity toward 1.
