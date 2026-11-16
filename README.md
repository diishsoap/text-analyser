# CLEARText — Text Complexity Analyser


**Track:** AI + Languages <br>
**Team:** Kiran Dhara (solo)

<br>


## The Problem
Often, high school students and undergraduates have to write reports, articles, essays, etc. on a certain academic topic. Sometimes, these articles and essays are published in journals which are publicly available to a non-specialised audience. The problem arises when these articles are far too difficult or technical for this audience to comprehend.


## What I Built
I believe knowledge should have no restrictions or barriers; so, I built an AI Natural Language Processing (NLP) model to help the user identify the complexity of their text. The user submits their text, and they receive a predicted complexity score, their relative percentile, an estimated difficulty (from three possible difficulties) and the word count. These statistics and results can then be further used to simplify or refine one’s text.


## How It Works
**Data:**

CLEAR (CommonLit Ease of Readability) Corpus: Crossley, S., Heintz, A., Choi, J.S. et al. A large-scale corpus for assessing text readability. Behav Res 55, 491–507 (2023). https://doi.org/10.3758/s13428-022-01802-x 

There are 3194 records in the processed data (4724 records in the original) and 40 different attributes of each record. 

The dataset was relatively clean, so I didn’t have to do much. I went through the license agreements of each record and eliminated all the records that didn’t have public domain (PD) listed for their license. 

Next, I checked for missing values and duplicates. The only missing values were in the optional metadata that I wasn’t going to use, and I found no duplicate IDs or excerpts across the whole dataset, so I didn’t change anything. Then, I standardized all whitespace characters as a single space (“ “) so that they don’t interfere with the training.

I chose this dataset because it included an attribute called 'BT Easiness', derived from teachers' judgments of which of two passages would be easier for a student to understand. These pairwise comparisons were used to calculate relative easiness scores using a Bradley–Terry model. This attribute aligned particularly well with my project's objectives. 

<br>

**Model:**

The baseline was trained at only the average BT Easiness score of the training section of the dataset. This lay down a very simple foundation that my model had to beat.

I trained my 1st model on a 80:20 training and testing split (using GroupShuffleSplit), using TF-IDF and ridge-regression. 

TF-IDF turned the text into numerical values, followed by Ridge Regression, which learned the relationship between those features and human-derived BT Easiness scores. I then combined both steps into a Scikit-learn pipeline to ensure consistent preprocessing during training and prediction.

I then developed a second model by replacing word-level TF-IDF with character-level TF-IDF, while retaining Ridge Regression. This allowed the model to analyse sequences of 3–5 consecutive characters rather than complete words and word combinations. This led to a modest increase in accuracy at the extremes.


## Results
Model 1 demonstrated a substantial improvement over the mean baseline, reducing MAE by approximately 35.9% and RMSE by 34.3%. However, error analysis revealed that it struggled particularly with passages at the extremes of the difficulty distribution, tending to overestimate the difficulty of easier passages and underestimate that of harder passages.
Model 2 achieved a further, modest improvement: MAE decreased by 1.73%, RMSE by 3.23%, and R² increased by approximately 4.81% relative to Model 1. Its error analysis also showed improved predictions for the most difficult and easiest passages, although performance declined slightly for the two middle difficulty groups.
Additionally, the standard deviation of its predictions increased from approximately 0.672 to 0.743, moving closer to the actual standard deviation of 1.029. This suggests that Model 2 captured a wider range of readability scores, although its predictions remained more concentrated around the mean than the actual scores.
In the end, Model 2 was selected as the preliminary model due to its better performance across all three evaluation metrics.

<br>

| Model | MAE | RMSE | R² |
| :--- | :---: | :---: | :---: |
| Baseline: mean BT Easiness score | 0.8383 | 1.0287 | -0.000 (zero deviation from mean / zero range) |
| Model 1: TF-IDF + ridge regression | 0.5375 | 0.6759 | 0.5682 |
| Model 2: + character level TF-IDF | 0.5282 | 0.6541 | 0.5955 |


The baseline's negative R² means it performed marginally worse than predicting the test-set mean. Its predictions also had zero variation because it predicted the same value every time. 




## Limitations

- **Prediction at extremes**: Both models tend to produce predictions closer to the mean, making particularly easy or difficult passages harder to estimate accurately.
- **Limited linguistic understanding**: The models primarily analyse character patterns and do not explicitly account for semantic complexity, contextual relationships, grammatical structures or figurative language.
- **Audience dependency**: The models do not account for the reader's age, educational background or prior knowledge, all of which can influence perceived readability.
- **Dataset limitations**: The training data consists of a limited collection of passages and teacher-derived judgements, which may not generalise equally well to all forms of writing.

Future development could incorporate additional linguistic features, such as average sentence length, vocabulary diversity and syllable count. More advanced language representations could also be explored to capture semantic and contextual complexity. 

Further testing on more diverse, previously unseen passages would help evaluate the model's generalisation and reliability.







## Sources
- CLEAR (CommonLit Ease of Readability) Corpus: Crossley, S., Heintz, A., Choi, J.S. et al. A large-scaled corpus for assessing text readability. Behav Res 55, 491–507 (2023). https://doi.org/10.3758/s13428-022-01802-x (Dataset is loaded into this notebook from a personal GitHub Repository, https://github.com/diishsoap/text-analyser. License agreement is mentioned in the README.)
- scikit-learn (TfidfVectorizer, Ridge, Pipeline, GroupShuffleSplit), pandas, NumPy, re, files, DummyRegressor, mean_absolute_error, mean_squared_error, r2_score,
- AI tools: I used ChatGPT to help debug, understand documentation of scikit-learn, to understand the flow of an AI project and required libraries, and to understand the methodology for the CLEAR dataset.

