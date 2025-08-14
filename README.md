Of course. Here is a more detailed and professional `README.md` file, breaking down the project's methodology, technical details, and potential future work.

---

# Twitter Sentiment Analysis Engine

![Python Version](https://img.shields.io/badge/Python-3.9+-blue.svg)
![Libraries](https://img.shields.io/badge/Libraries-Pandas%20%7C%20NumPy-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

This repository contains the code and resources for a sentiment analysis project completed for the **DSAI 1303: Data Science Programming Languages** course. The project tackles the challenge of quantifying public opinion by processing and analyzing a dataset of tweets. It involves data cleaning, sentiment scoring based on a lexicon, and a novel approach to derive the sentiment of previously unknown terms.

## 🗂️ Table of Contents
- [Project Overview](#-project-overview)
- [Technical Walkthrough](#-technical-walkthrough)
  - [Part 1: Data Loading & Preprocessing](#part-1-data-loading--preprocessing)
  - [Part 2: Calculating Tweet Sentiment](#part-2-calculating-tweet-sentiment)
  - [Part 3: Deriving New Term Sentiment](#part-3-deriving-new-term-sentiment)
- [Project Structure](#-project-structure)
- [How to Run This Project](#-how-to-run-this-project)
- [Outputs Explained](#-outputs-explained)
- [Future Improvements](#-future-improvements)

## 🎯 Project Overview

The core objective is to build a system that can automatically assign a sentiment score to text from Twitter. This is achieved through a three-step pipeline:

1.  **Load and Clean:** Ingest raw tweet data and apply a series of text-cleaning operations to remove noise and normalize the text for analysis.
2.  **Estimate Tweet Sentiment:** Calculate a sentiment score for each tweet by summing the scores of its individual words, using the `AFINN-111` lexicon.
3.  **Estimate Term Sentiment:** For words not present in the initial lexicon, deduce their sentiment based on the overall sentiment of the tweets in which they appear.

## 🛠️ Technical Walkthrough

This section details the logic implemented in the `Sentiment Analysis.ipynb` notebook.

### Part 1: Data Loading & Preprocessing

The foundation of any text analysis is clean, high-quality data. This step transforms raw, noisy tweets into a usable format.

**Input:** A raw data file (`tweets.json` or `*.csv`) containing tweet text.

**Process:**
A custom function applies a series of regular expressions and string methods to each tweet:
1.  **Remove User Mentions:** Mentions like `@username` are removed as they typically don't contribute to sentiment.
    ```python
    re.sub(r"@\S*", " ", tweet_text)
    ```
2.  **Remove URLs:** Hyperlinks are stripped out, as they are external references, not direct expressions of sentiment.
    ```python
    re.sub(r"https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\\+.~#?&//=]*)", " ", tweet_text)
    ```
3.  **Convert to Lowercase:** The entire text is converted to lowercase to ensure uniformity (e.g., 'Happy', 'happy', and 'HAPPY' are treated as the same word).
    ```python
    tweet_text.lower()
    ```
4.  **Strip Extraneous Punctuation:** Punctuation marks at the beginning or end of words are removed to isolate the core term.
    ```python
    word.strip("’.,;%$&#!'?")
    ```

**Example:**
-   **Before:** `@someuser Check out this amazing project, it's so cool! https://example.com #datascience`
-   **After:** `check out this amazing project, it's so cool`

**Output:** A file named `clean_tweets.txt` where each line contains one cleaned tweet.

### Part 2: Calculating Tweet Sentiment

With clean data, we can now quantify the sentiment of each tweet.

**Input:**
-   `clean_tweets.txt` (from Part 1)
-   `AFINN-111.txt`: A sentiment lexicon file where each line contains a term and its corresponding sentiment score (from -5 to +5), separated by a tab.

**Process:**
1.  **Load Lexicon:** The `AFINN-111.txt` file is parsed into a Python dictionary, mapping each term to its integer score.
    ```python
    scores = {'abandon': -2, 'abandoned': -2, ..., 'zeal': 3}
    ```
2.  **Score Each Tweet:** For every tweet in `clean_tweets.txt`:
    -   Initialize a total score variable: `tweet_score = 0`.
    -   Split the tweet into individual words.
    -   For each word, look it up in the `scores` dictionary.
        - If the word exists, add its score to `tweet_score`.
        - If the word does not exist, its score is considered **0**.
    -   The final `tweet_score` is the sum of all its word scores.

**Formula:**
`Tweet_Sentiment = Σ (Score_of_word_i)` for all words `i` in the tweet.

**Output:** A file named `sentiment.txt` containing one numerical score per line, corresponding to each tweet in `clean_tweets.txt`.

### Part 3: Deriving New Term Sentiment

This is the most innovative part of the project: determining the sentiment of words that are *not* in the AFINN lexicon. The underlying principle is **"sentiment by association."**

**Problem:** How do we score a word like "football" if it doesn't exist in `AFINN-111.txt`?

**Process:**
1.  **Identify Unknown Terms:** First, a list of all words that appear in the tweets but not in the AFINN dictionary is created.
2.  **Filter Noise:** This list is filtered to remove common, non-sentimental words (stop words) and very short, likely meaningless tokens.
3.  **Associate Terms with Tweet Scores:** The system iterates through every tweet and its pre-calculated sentiment score (from Part 2). If a tweet contains an "unknown term," a pairing is created: `(unknown_term, tweet_score)`.
4.  **Aggregate Scores:** Since an unknown term can appear in many tweets, we need a final score. The project uses a `groupby()` operation on the term, followed by a `sum()` of all associated tweet scores. This aggregates the sentiment context in which the word appears.

**Conceptual Example:**
If "football" appears in three tweets with scores `+3` (e.g., "I love football"), `-2` (e.g., "football is so boring"), and `+4` (e.g., "what a great football game!"), its derived sentiment would be:
`Derived_Score('football') = (+3) + (-2) + (+4) = 5`

**Output:** A list or DataFrame mapping each new term to its calculated sentiment score, effectively expanding our sentiment lexicon.

## 📁 Project Structure
```
.
├── Sentiment Analysis.ipynb    # Main Jupyter Notebook with all code and explanations.
├── tweets.json                 # Raw input data file with tweets (or .csv equivalent).
├── AFINN-111.txt               # The sentiment lexicon file.
├── clean_tweets.txt            # (Output) Cleaned tweets, one per line.
├── sentiment.txt               # (Output) The final sentiment score for each tweet.
└── README.md                   # This file.
```

## 🚀 How to Run This Project

### Prerequisites
-   Python 3.9 or higher
-   Jupyter Notebook or JupyterLab
-   The following Python libraries: `pandas`, `numpy`

### Setup
1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```
2.  **Install dependencies:**
    ```bash
    pip install pandas numpy jupyterlab
    ```
3.  **Prepare Data:**
    - Place `tweets.json` (or your CSV data file) and `AFINN-111.txt` in the root of the project directory.
    - **Important:** If using the provided notebook, you may need to update the file path in the `pd.read_csv()` function to match the location of your data file.

### Running the Analysis
1.  **Start JupyterLab:**
    ```bash
    jupyter lab
    ```
2.  **Open and Run the Notebook:**
    - Open the `Sentiment Analysis.ipynb` file.
    - Execute the cells sequentially from top to bottom. The notebook is designed to be run in order, as later cells depend on the outputs of earlier ones.

## 📊 Outputs Explained

-   **`clean_tweets.txt`**: A text file containing the processed tweets. Each line is a single tweet, stripped of mentions, URLs, and excess punctuation, and converted to lowercase.
-   **`sentiment.txt`**: A text file containing a single integer or float on each line. The number on line `N` is the calculated sentiment score for the tweet on line `N` of `clean_tweets.txt`.
-   **New Term Sentiments (In-Notebook Output)**: The final cells of the notebook will display a Pandas DataFrame showing the newly discovered terms and their derived sentiment scores.

## 🔮 Future Improvements
This project provides a solid foundation. Future work could include:
-   **Advanced Preprocessing:** Implement lemmatization or stemming to group different forms of a word (e.g., "run," "running," "ran") into a single token.
-   **Handling Negations and Modifiers:** The current model doesn't handle negations (e.g., "not good") or intensifiers (e.g., "very good") effectively. A more advanced parser could be used to invert or amplify scores based on these modifiers.
-   **Using More Sophisticated Models:** Explore machine learning models like Naive Bayes, SVM, or deep learning models like LSTMs or Transformers (e.g., BERT) for sentiment classification, which can learn patterns from the data itself.
-   **Interactive Dashboard:** Build a simple web application using Flask or Streamlit to allow users to input a tweet and see its sentiment score in real-time.
