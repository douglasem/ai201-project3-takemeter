# TakeMeter

## Community
I chose r/Parenting as the community for this project. Parenting discussions contain a mix of personal stories, practical advice, and emotional support, making them a good fit for a text classification task. The distinctions between these types of responses matter because parents often visit the community looking for specific kinds of help, whether that is actionable guidance, shared experiences, or encouragement.


## Labels
Advice
A post or comment whose primary purpose is recommending an action, strategy, or solution for another parent.

Examples:
* “Talk with the teacher before deciding whether to delay kindergarten.”
* “Try moving bedtime 30 minutes earlier and see if it helps.”

Experience
A post or comment whose primary purpose is sharing a personal parenting experience, story, or observation.

Examples:
* “We delayed kindergarten for our son and it ended up being the right choice.”
* “My daughter stopped taking naps around age three.”

Support
A post or comment whose primary purpose is providing empathy, encouragement, validation, or emotional support.

Examples:
* “You’re doing a great job. Parenting is hard.”
* “That sounds incredibly stressful. Hang in there.”


## Hard Edge Cases
The most difficult boundary is between Advice and Experience.

Example 1:
“We delayed kindergarten for our son and it helped tremendously. I’d recommend considering that option.”

This comment contains both a personal story and a recommendation.

Decision rule:
If the primary purpose is recommending an action to another parent, label it Advice.
If the primary purpose is sharing what happened to the writer or their child, label it Experience.

Example 2:
“My little guy was playing with a group of kids at a party… Used the situation to remind him…”

Could be:
* Advice
* Experience

Decision:
* Experience

Reason:
* Primary purpose is recounting what happened.

Example 3:
“She did an excellent job. I always told my son to stay put and I will find him.”

Could be:
* Support
* Advice

Decision:
* Support

Reason:
* Primary purpose is praising and reassuring OP.

Example 4:
“It gets incrementally easier every day. Think back to the second month with a newborn and compare it to now.”

Could be:
* Support
* Advice

Decision:
* Advice

Reason:
* Gives a concrete reframing strategy.


## Data Collection Plan
I will collect at least 200 public posts and comments from r/Parenting.

My target distribution is:
- Advice: ~65 examples
- Experience: ~65 examples
- Support: ~65 examples

If one label becomes underrepresented, I will intentionally collect additional examples matching that category until the distribution is reasonably balanced.



## Evaluation Metrics
I will evaluate the classifier using:
- Overall Accuracy
- Precision
- Recall
- F1 Score for each label
- Confusion Matrix

Accuracy alone is not sufficient because a model could perform well on a majority class while performing poorly on other labels. Per-class metrics and a confusion matrix will help identify specific label boundaries that are difficult for the model.


## Definition of Success
The classifier will be considered successful if it achieves:
- At least 75% overall accuracy
- At least 0.70 F1 score for each label

This level of performance would be useful for organizing parenting discussions by intent and helping users quickly find advice, experiences, or emotional support.


## AI Tool Plan
### Label Stress Testing
I will use ChatGPT to generate borderline parenting comments that sit between Advice and Experience to verify that my label definitions are clear.

### Annotation Assistance
I may use ChatGPT to suggest labels for batches of examples, but I will manually review every example before finalizing the dataset.

### Failure Analysis
After evaluation, I will use ChatGPT to help identify patterns in the model's incorrect predictions and then verify those patterns manually.