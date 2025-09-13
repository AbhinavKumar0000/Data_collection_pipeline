# Job Posting Data Pipeline: Collection, Cleaning, and Annotation

## Overview

This project implements a three-stage data pipeline to collect, clean, and annotate remote software development job postings. The pipeline starts by fetching raw data from the Remotive API, proceeds to clean and standardize the job descriptions, and finally performs Exploratory Data Analysis (EDA) to annotate each posting with structured labels such as experience level, job type, and relevant technical skills.

The final output is a structured dataset, available in both CSV and JSON formats, ready for further analysis, model training, or use in a job aggregation platform.

---

## Methodology and Approach

The pipeline is structured into three sequential Python scripts, each performing a distinct task.

### Stage 1: Data Collection (`Scraper_script.ipynb`)

-   **Objective**: To fetch raw job posting data from an external source.
-   **Approach**: The script connects to the Remotive REST API, specifically targeting the "software-development" category.
-   **Execution**:
    1.  A GET request is sent to the API endpoint (`https://remotive.com/api/remote-jobs?category=software-development&limit=50`).
    2.  The JSON response containing a list of jobs is parsed.
    3.  For each job, the `job_description` field, which contains HTML content, is parsed using **BeautifulSoup** to extract clean, plain text.
    4.  The relevant fields (`job_title`, `company_name`, `location`, `job_description`, `url`) are mapped into a structured format.
    5.  The collected data is compiled into a pandas DataFrame and saved as `raw_data.csv`.

### Stage 2: Data Cleaning (`Cleaning_script.ipynb`)

-   **Objective**: To preprocess the raw text data to make it suitable for analysis and annotation.
-   **Approach**: This script focuses on standard text normalization and data sanitization.
-   **Execution**:
    1.  The `raw_data.csv` file is loaded into a pandas DataFrame.
    2.  Duplicate entries are removed based on the unique `url` for each job posting.
    3.  Records with null `job_description` values are dropped.
    4.  A custom cleaning function is applied to the `job_description` column, which performs the following using regular expressions (`re`):
        -   Converts all text to lowercase.
        -   Removes all non-alphanumeric characters (except spaces).
        -   Collapses multiple whitespace characters into a single space.
    5.  The processed DataFrame, now containing a `cleaned_description` column, is saved as `cleaned_data.csv`.

### Stage 3: Exploratory Data Analysis & Annotation (`Annotation_script.ipynb`)

-   **Objective**: To analyze the cleaned text to identify key patterns and enrich the dataset with structured tags.
-   **Approach**: This stage uses NLP techniques for EDA and a rule-based system for annotation.
-   **Execution**:
    1.  **Exploratory Data Analysis (EDA)**:
        -   The `cleaned_data.csv` is loaded.
        -   Scikit-learn's `CountVectorizer` is used to perform frequency analysis on both single words (unigrams) and two-word phrases (bigrams). English stop words are removed to filter out noise.
        -   The top 20 most frequent unigrams and bigrams are visualized using **Matplotlib** and **Seaborn** to identify common terms and skills.
    2.  **Dynamic Skill List Generation**:
        -   The frequent terms from the EDA are cross-referenced with a predefined `MASTER_SKILL_LIST` of known technical skills.
        -   This creates a smaller, contextually relevant `SKILL_KEYWORDS` list tailored to the dataset, improving the efficiency and accuracy of skill tagging.
    3.  **Annotation**:
        -   Three functions are defined to apply rule-based labeling:
            -   `get_experience_level()`: Scans the description for keywords like "senior," "lead," "jr," or years of experience (e.g., "5+ years") to categorize the job's seniority.
            -   `get_job_type()`: Looks for terms like "backend," "frontend," "full stack," or "devops" to classify the job specialization.
            -   `extract_skill_tags()`: Iterates through the dynamic `SKILL_KEYWORDS` list and tags the job with any skills mentioned in its description.
    4.  **Output**:
        -   The annotations are applied to the DataFrame, creating new columns: `experience_level`, `job_type`, and `skill_tags`.
        -   A final, annotated sample of 20 records is saved to `annotated_data.csv` and `annotated_data.json`.

---

## Tools and Libraries Used

-   **Data Handling**: `pandas`, `numpy`
-   **Web Scraping & API**: `requests`, `bs4` (BeautifulSoup)
-   **Text Processing**: `re` (Regular Expressions)
-   **NLP & Analysis**: `scikit-learn` (for `CountVectorizer`)
-   **Data Visualization**: `matplotlib`, `seaborn`

---

## How to Run the Pipeline

To execute the pipeline, run the Jupyter notebooks in the following order. Ensure all required libraries are installed.

1.  **Run `Scraper_script.ipynb`**: This will fetch the data from the Remotive API and create `raw_data.csv`.
2.  **Run `Cleaning_script.ipynb`**: This will process `raw_data.csv` and generate the `cleaned_data.csv` file.
3.  **Run `Annotation_script.ipynb`**: This will perform EDA on the cleaned data and produce the final `annotated_data.csv` and `annotated_data.json` files.

---

## Challenges Faced

1.  **Rule-Based Annotation Limitations**: The annotation for experience level and job type relies on a predefined set of keywords. This approach is simple and fast but may not capture all nuances. For instance, a job might be "Senior" without explicitly using the word, or a "Full-Stack" role might be described without the exact phrase. A more advanced approach could use machine learning models for classification.
2.  **HTML Parsing**: Job descriptions fetched from the API contained HTML tags. While BeautifulSoup is effective, poorly structured or inconsistent HTML could lead to improperly cleaned text. The approach of stripping all tags and joining text with spaces was a robust general solution.
3.  **Skill Identification**: Relying solely on a master list of skills could be inefficient and lead to tagging irrelevant skills. The implemented two-step process—first using EDA to find frequent terms and then filtering them against a master list—creates a more relevant set of keywords for tagging, but its effectiveness is dependent on the quality and size of the initial dataset.

---

## Output Files

This pipeline generates the following files:

-   `raw_data.csv`: The initial, unprocessed data collected from the API.
-   `cleaned_data.csv`: The data after deduplication, handling nulls, and text normalization.
-   `annotated_data.csv`: A sample of 20 records with added columns for experience level, job type, and skill tags.
-   `annotated_data.json`: The same annotated sample in JSON format.
