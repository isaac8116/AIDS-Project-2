# AIDS-Project-2

## Overview

This project is a completely automated multi-stage pipeline that churns out detailed reports on any product category. The stages are:

1. Product Description
2. Data Scraping
3. Data Analysis
4. Design Requirement Synthesis

## Folder Structure

- `old_analyze/`: Contains the old version of the analysis code.
- `old_scrape/`: Contains the old version of the scraping code.
- `session/{PRODUCT}/`: The session folder for a product category where all intermediate data files are stored.
- `wireless over-ear headphones.zip`: The zipped session folder (the data basically) for the generated report we mentioned in the report.

### Notebooks

All intermediate data is stored into the `session/{PRODUCT}/` session folder.

- `1_describe_product.ipynb`: From product category, outputs design metrics, competitors and common tech specs.
- `2_find_datasheet.ipynb`: Given competitors, searches for datasheet webpage and outputs their cleaned HTML.
- `2_scrape_reddit_reviews.ipynb`: Given competitors, searches and scrapes Reddit comments.
- `2_youtube_search.ipynb`: Give competitors, searches for Youtube videos like reviews, comparisons, etc.
- `2b_scrape_youtube_comments.ipynb`: Given Youtube video IDs, scrapes the comments.
- `3_extract_specs.ipynb`: Extracts the specs from the datasheet HTML earlier.
- `3_gemini_video_analysis.ipynb`: Given Youtube video IDs, fully transcribes the video then extracts feedback.
- `3_reddit_comment_analysis.ipynb`: Given Reddit comments, calculate rating for each metric.
- `3_youtube_comment_analysis.ipynb`: Given Youtube comments, calculate rating for each metric.
- `3b_summarize_comments.ipynb`: Combines all comments and extracts common feedback.
- `4_synergize_insights.ipynb`: Combines feedback from videos and comments, metrics, and tech to generate design requirements report PDF.

## Guide

To make things easier to understand, follow along the report as you read the code, as the report is structured in the same order as the code while providing a high-level overview of each step. Contrary to what I said in the report, the report is probably the better guide to follow.

```sh
# (Optional) Create a virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

Now, use your notebook IDE of choice such as `jupyter lab` or VSCode. Before we continue, obtain the following API keys:

- Gemini AI Studio API Key: <https://ai.google.dev/gemini-api/docs/api-key>
  - With the default settings, all notebooks can run without exceeding the free tier requests per day.
- Google Cloud API Key with YouTube Data API enabled: <https://developers.google.com/youtube/v3/getting-started>
- Google Cloud API Key with Custom Search enabled: <https://developers.google.com/custom-search/v1/overview>
- Custom Search Engine ID: <https://programmablesearchengine.google.com/controlpanel/all>
  - Ensure that "Search the entire web" is enabled in the settings.
- Reddit Credentials for PRAW: <https://praw.readthedocs.io/en/stable/getting_started/authentication.html>

In each notebook, near the top, you will find the code cell to set API keys. For example:

```python
# 2_scrape_reddit_reviews.ipynb

# Reddit API credentials
reddit = praw.Reddit(
    client_id='RhCT2FHyjBBd1AjbnqylMQ',
    client_secret='A_wD-PpvLqLY9w-8VgB54hzkbHSuYA',
    user_agent='Izowk'
)
```

```python
# 3_gemini_video_analysis.ipynb

# Should be from Google AI Studio.
GOOGLE_AI_KEY = "AIzaSyDAlPx7St5BUXqlwiqFKvlT-Sc2dnTT4Jc"
# 2.5 Flash is good tradeoff between better vid understanding and free 500 RPD.
GOOGLE_AI_MODEL = "gemini-2.5-flash-preview-04-17"
# GOOGLE_AI_MODEL = "gemini-2.5-pro-exp-03-25"
```

The keys we used remain in the Git history, but don't worry, we already invalidated all API keys used in this project.

Likewise, settings are located near the top of each notebook right after imports under the "Config" section heading. For example:

```python
# 1_describe_product.ipynb

PRODUCT = "wireless over-ear headphones"
NUM_METRICS = 5
NUM_COMPETITORS = 5

# Of lesser importance; used for 3_extract_specs.ipynb due to poor accuracy of direct extraction.
MAX_SPECS = 10
```

The most important config to take note of is `PRODUCT`. This determines the session folder used, so do check its correct to avoid confusion. Along those lines, to restart analysis from the beginning, you have to move or delete the `session/{PRODUCT}/` session folder.

With that, run the notebooks in order. This is primarily determined by the first number in the filename which corresponds to which stage of the pipeline it is in. Within the same stage, notebooks can be run in any order, except for `2b` or `3b` which are dependent on prior notebooks within the same stage.

When running, it is strongly recommended not to use the "Run All" button. The cells are meant to be followed along, and will display debug output that is useful to understand the code. If you use "Run All", it is pretty overwhelming. Furthermore, this makes it easier to tell if something is going wrong (not that there should be any major issues)

## Session Folder Structure

Under each `session/{PRODUCT}/` session folder generated for a given input product category, the following files are present at the end of the pipeline:

- `comment_analysis/`: Contains the extracted feedback from combining Youtube and Reddit comments.
  - `{COMPETITOR_NAME}/{METRIC_NAME}.json`: The extracted feedback for each metric. Note that competitors and metrics are dynamically generated in stage 1.
- `reddit/`: Contains the scraped Reddit comments.
  - `processed_comments/{COMPETITOR_NAME}.csv`: Contains the processed comments after sentiment analysis, zero-shot topic classification, and filtering.
  - `raw_comments/{COMPETITOR_NAME}.csv`: Contains the raw comments scraped from Reddit.
  - `competitor_map.json`: Map from verbatim competitor names to actual file names.
  - `feature_summary.csv`: Contains the rating for each metric.
- `youtube/`: Contains the scraped Youtube comments.
  - `processed_comments/{COMPETITOR_NAME}.csv`: Contains the processed comments after sentiment analysis, zero-shot topic classification, and filtering.
  - `raw_comments/{COMPETITOR_NAME}.csv`: Contains the raw comments scraped from Youtube.
  - `comment_filemap.json`: Map from verbatim competitor names to actual file names.
  - `feature_youtube_comment_summary.csv`: Contains the rating for each metric.
  - `vid_meta.json`: Contains video IDs for each video type for each competitor.
  - `transcripts.json`: Contains all video transcripts.
  - `analysis.json`: Contains feedback extracted from video + transcripts for all competitors.
- `datasheets.json`: Contains cleaned HTML datasheets for each competitor.
- `specs.json`: Contains the extracted specs for each competitor.
- `stage_1.json`: Contains the output of stage 1, including competitors, metrics, and tech specs.
- `report.pdf`: The final report containing design requirements generated from the analysis.
