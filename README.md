# Parenting Comment Intent Classification

## Project Overview
This project fine-tunes a text classification model to identify the primary intent of comments in the Reddit parenting community.

The goal is to classify comments into one of three categories:

* advice – recommending an action, strategy, or solution
* experience – sharing a personal story, observation, or parenting experience
* support – providing empathy, encouragement, validation, or emotional support

The project compares a zero-shot large language model baseline against a fine-tuned DistilBERT model trained on a manually labeled dataset.


## Community Choice and Reasoning
I selected the Reddit parenting community because it contains a high volume of discussions where parents exchange advice, personal experiences, and emotional support.
These three comment types frequently appear together in the same threads but represent different communication goals, making them a good candidate for intent classification.
The task is challenging because the labels are based on the purpose of the comment rather than the topic being discussed.


## Repository Contents
- planning.md
- README.md
- data/parenting_discussions.csv
- outputs/evaluation_results.json
- outputs/confusion_matrix.png


## Dataset
The labeled dataset is included in this repository:
`data/parenting_discussions.csv`


## Label Taxonomy
### Advice
A comment whose primary purpose is recommending an action, strategy, or solution for another parent.

Examples:
Talk with the teacher before deciding whether to delay kindergarten.
Try moving bedtime 30 minutes earlier and see if it helps.

### Experience
A comment whose primary purpose is sharing a personal parenting experience, story, or observation.

Examples:
We delayed kindergarten for our son and it ended up being the right choice.
My daughter stopped taking naps around age three.

### Support
A comment whose primary purpose is providing empathy, encouragement, validation, or emotional support.

Examples:
You’re doing a great job. Parenting is hard.
That sounds incredibly stressful. Hang in there.


## Data Collection and Labeling
Data was collected from publicly available Reddit comments in r/Parenting discussion threads. No private or authenticated content was used.

A total of 218 comments were manually reviewed and labeled.

Each example was labeled according to the primary purpose of the comment rather than the topic being discussed.

The dataset was saved as a CSV with the following schema:

text,label,notes

The dataset was automatically split by the notebook into:
Train: 152 examples
Validation: 33 examples
Test: 33 examples


## Label Distribution
| Label | Count |
|--------|------:|
| Experience | 115 |
| Advice | 60 |
| Support | 43 |
| Total | 218 |

No label exceeded the 70% imbalance threshold required by the project specification.


### Baseline Prompt
The zero-shot baseline used the following label definitions:

- advice: recommending an action, strategy, or solution
- experience: sharing a personal parenting experience or observation
- support: providing empathy, encouragement, or validation

The model was instructed to return only one label:
advice, experience, or support.

Each test example was sent individually to the zero-shot model, which returned one of the three labels. The notebook then compared those predictions against the held-out test labels to calculate accuracy and per-class metrics.


## Difficult Labeling Decisions
### Example 1
It gets incrementally easier every day. Think back to the second month with a newborn and compare it to now.

Possible labels:
* Advice
* Support

Final label:
* Advice

Reason:
The primary purpose is helping another parent understand what to expect and how to interpret their situation.


### Example 2
I know it’s so cliché, but it does get better. What helps is finding something I loved about each stage.

Possible labels:
* Advice
* Support

Final label:
* Advice

Reason:
Although emotionally supportive, the comment ultimately recommends a perspective and coping strategy.


### Example 3
We delayed kindergarten for our son and it helped tremendously. I’d recommend considering that option.

Possible labels:
* Experience
* Advice

Final label:
* Advice

Reason:
The personal story supports a recommendation, but the recommendation is the primary purpose of the comment.


## Fine-Tuning Approach
Base Model
* DistilBERT (distilbert-base-uncased)

Training Configuration
* Epochs: 3
* Learning Rate: 2e-5
* Batch Size: 16

Default notebook hyperparameters were used.
The model was trained using Google Colab with a T4 GPU.


## Baseline
The baseline was generated using the Groq API with the llama-3.3-70b-versatile model as a zero-shot classifier.

The prompt included:
* community description
* label definitions
* one example per label
* instruction to output only a label name

The baseline was evaluated on the locked test set before fine-tuning.

### Baseline Accuracy
**Accuracy:** 0.848

Baseline Per-Class Metrics
| Label | Precision | Recall | F1 |
|---------|---------:|------:|------:|
| Advice | 1.00 | 0.60 | 0.75 |
| Experience | 0.84 | 1.00 | 0.91 |
| Support | 0.75 | 0.86 | 0.80 |


### Fine-Tuned Model Accuracy
**Accuracy:** 0.485

Fine-Tuned Per-Class Metrics
| Label | Precision | Recall | F1 |
|---------|---------:|------:|------:|
| Advice | 0.00 | 0.00 | 0.00 |
| Experience | 0.48 | 1.00 | 0.65 |
| Support | 0.00 | 0.00 | 0.00 |


## Model Comparison
| Model | Accuracy |
|---------|---------:|
| Zero-Shot Groq Baseline | 0.848 |
| Fine-Tuned DistilBERT | 0.485 |

The fine-tuned model underperformed the baseline by approximately 36 percentage points.


## Fine-Tuned Confusion Matrix
| True Label | Predicted Advice | Predicted Experience | Predicted Support |
|------------|----------------:|---------------------:|------------------:|
| Advice | 0 | 10 | 0 |
| Experience | 0 | 16 | 0 |
| Support | 0 | 7 | 0 |

The model predicted Experience for every test example.

This represents a collapse into a single-class prediction strategy.


## Error Analysis
After reviewing the misclassified examples and using an AI tool to identify patterns, I found that nearly all errors involved predicting Experience for comments that were actually Advice or Support.

The AI tool suggested several possible explanations:
* Comments containing personal pronouns (“my son”, “my kids”, “when mine were”) were frequently interpreted as Experience.
* Very short supportive comments lacked explicit emotional keywords.
* Advice comments often contained personal stories that resembled Experience examples.

I verified these patterns manually by reviewing the misclassified examples.


### Failure Example 1
Text
Turn it into a song. My kids learned my mobile number by singing it to the tune of Mary Had A Little Lamb.

True Label: Advice
Predicted: Experience

Analysis
The comment contains both a recommendation and a personal story. According to the labeling rules, recommendations take priority when they are the primary purpose of the comment.
The model appears to focus on the phrase:
My kids learned…
which resembles many Experience examples from the training set.
This suggests the model learned surface-level language patterns rather than communicative intent.


### Failure Example 2
Text
Tf is me time?

True Label: Support
Predicted: Experience

Analysis
This comment is intended as humor and solidarity with another parent. It is not sharing a story.
The model likely struggled because support comments were often short and lacked explicit emotional words such as “sorry,” “hang in there,” or “you’re doing great.”


### Failure Example 3
Text
There’s a censored version called Taskmaster Bleeped.

True Label: Advice
Predicted: Experience

Analysis
The comment provides useful guidance to another parent.
However, it does not contain obvious recommendation language such as:
* try
* consider
* you should

The model may have learned to associate advice with explicit recommendation phrases and failed when advice was expressed indirectly.


## Sample Classifications
| Text | Prediction | Confidence |
|------|------------|-----------:|
| "My daughter got separated from me at Disney World as well..." | Experience | 0.41 |
| "We put an AirTag on my kid at Disney." | Experience | 0.40 |
| "My son at 6 lost us and ended up riding Splash Mountain..." | Experience | 0.42 |
| "Tf is me time?" | Experience | 0.36 |
| "Turn it into a song. My kids learned my mobile number..." | Experience | 0.37 |

Correct Prediction Example
My daughter got separated from me at Disney World as well…
The prediction of Experience is reasonable because the comment primarily recounts a personal story and observation.


## Reflection
The model learned a much simpler decision boundary than intended.

The label definitions focused on communicative intent:
* giving advice
* sharing experiences
* offering support

However, the model appeared to rely heavily on surface-level patterns instead.

Comments containing phrases such as:
* my son
* my daughter
* my kids
* when mine were younger

were frequently classified as Experience even when they were primarily giving advice.

Likewise, many short support comments were classified as Experience because they lacked explicit emotional keywords.

The results suggest that intent classification is substantially harder than topic classification when only a few hundred training examples are available.

The large language model baseline already possessed strong knowledge of conversational intent and significantly outperformed the fine-tuned DistilBERT model.


## Spec Reflection
How the Specification Helped
The specification required establishing a baseline before training. This was extremely useful because it provided context for interpreting the fine-tuned model’s performance. Without the baseline, I might have considered 48.5% accuracy acceptable. The baseline showed that a general-purpose language model could already achieve 84.8% accuracy on the task.

How My Implementation Diverged
The original goal was to improve upon the baseline through fine-tuning. Instead, the fine-tuned model performed substantially worse. Rather than iterating on additional hyperparameter tuning or collecting a much larger dataset, I kept the implementation aligned with the baseline project requirements and documented the failure mode as part of the evaluation.


## AI Usage
Example 1: Annotation Assistance
I used ChatGPT to help pre-label batches of Reddit comments according to the label definitions.
However, every example was manually reviewed before being added to the dataset.
Several labels were changed during review, particularly Advice versus Experience edge cases.


Example 2: Error Analysis
After obtaining the fine-tuned model results, I provided misclassified examples to ChatGPT and asked it to identify common patterns.

The tool suggested that:
* Advice comments containing personal stories resembled Experience examples.
* Short support comments lacked strong support keywords.
* Experience was acting as a default prediction category.

I reviewed the examples manually and confirmed these patterns before including them in the evaluation report.