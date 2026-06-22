# TakeMeter
TakeMeter is a a fine-tuned text classifier that evaluates player transfer discourse quality in the Tottenham Hotspur Football Club fan subreddint r/coys.

## Community
As a lifelong fan of Tottenham Hotspur Football Club, I am deeply familiar with the online discourse surrounding player transfer news, which led me to choose the club's subreddit, r/coys, for this project. Sorting the wheat from the chaff to find truly informed perspectives on potential incoming transfers is often a challenge requiring constant scrolling to find the few nuggets of genuine insight. Discourse quality in this community leaves detectable signals in the text itself. Informed posts tend to contain named entities (people, clubs, leagues), quantitative claims (stat comparisons), tactical vocabulary (positions and formations), and evidential language (specific first-hand viewing experiences). Because these distinct textual markers separate high-quality analysis from standard fan-narrative based speculation or low-effort reaction, they provide features for a text classifier to learn and detect. A model fine-tuned to classify this discourse could be used to develop a tool that surfaces only the most informed takes for a fans looking to get a quick read on the potential transfer.

## Label Taxonomy

### Informed
A post that tells the reader something they didn't already know or assume about the player, deal, or situation. It contains a specific detail, fact, or context — e.g. injury history, prior transfer links, performance in a different league/system, club history, cross-fanbase sentiment, or financial/regulatory context. Pure opinion, prediction, or reaction with no new informational content belongs in Standard.

**Examples**
> "eh that was mostly Kwasniok waffling imo. He got most of his g+a in starts (11 in 20 starts). After Kwasniok was sacked he started every game and still got 3 goals and an assist in the last 7 games. His record of the bench was impressive but I think thats just cause he is a real good player and will get things done no matter what"

> "He was on a tear last season. He single-handedly saved Koln from relegation. Just watch highlights from their final game against Bayern for his goal to get a sense of his skill set. There was quite a stink that Nagelsmann didn't select him for the national team and with Sane instead."

### Standard
A post that contributes a coherent perspective rooted in football knowledge or fan context, but whose claims rest on narrative and belief rather than verifiable evidence or specific data. The post may appear specific, referencing real players, fees, or contract structures, but claims are speculative, hypothetical, or built from facts already known to an informed Spurs fan, used to support an opinion rather than to inform the reader of something new.

**Examples**

> "I feel we should buy two attackers anyways though, Koln accepted a roughly £40m bid for him from Brentford, if we add Savinho as well, we'll spend maybe just under £100m for both."

> "Loaning Vuskovic back to Hamburg is probably the ideal situation. Let him fulfill his dream playing with his brother and he will be grateful with TOT. Before he left make him sign a new 5-year deal with bumper wages and let Hamburg covers the first year. Assuming Romero and Drag left this window and Danso next then we will have him competing with VDV, PVH and Senesi hopefully for a CL challenging squad in 2028."

### Low-effort
A post that contributes nothing beyond an emotional reaction, headline restatement, or assertion so vague or unsupported that it adds no value to the discussion It could have been written by someone who only read the headline or requires no football knowledge, no familiarity with the player, and no analytical thought to produce.

**Examples**
> "Honestly a great signing and after the news that was coming out it’s cheaper than I was expecting"

> "I'm sure they like money more than they hate us"

## Data Collection

All data was collected from the Tottenham Hotspur Football Club subreddit r/coys, and strictly from threads related to player transfer rumors. Informed posts are genuinely rare relative to standard or low-effort posts, particularly in transfer-rumor threads which are often dominated by speculation and emotional reactions. I manually selected all informed posts from a range of 9 different threads. Given the scarcity of informed posts, these posts amount to 20% of the example mix. For standard and low-effort posts, I used `claude-sonnet-4.7` to label every comment from the same 9 threads using the label definitions (~1200 posts). From there, I prompted Claude to write a script to select 100 low-effort and standard posts at random. These were personally reviewed and overridden where necessary. Exceptionally short (< 5 words), posts containing links or images, or were deleted were removed from the list. The total example count was 204 posts with the following distribution.

**Label distribution:**
low-effort    84
standard      80
informed      40

### Difficult-to-label examples

> "I think Romero is very close to a top player and his ball playing ability has been severely underrated, but I'm more than happy to replace him with Van Hecke. Injuries and suspensions make him miss at least 1/3 of the season every year, also he seems to be an awful captain. Has also just not been that committed to us because he thinks he should be at a better club. In context (not just individual football quality) it's a pretty big upgrade imo, keeping Romero would mean trouble, especially with De Zerbi in charge."

This is borderline between Standard and Informed. The discourse seems grounded in first-person match viewing, and there is a reference to stat (misses 1/3rd) of the season which makes it lean Informed. But this is contrasted with narrative-based assumptions about the player's discipline history and potential personality clash with the new coach. If the comment about missing at least 1/3rd of the season is removed, the post reads much more Standard than Informed. I ultimately went with Standard.

> "52m for a guy with 1 year left on his contract is mad. But I will take it, what ever De Zerbi wants"

This is borderline between Standard and Low-effort because of the coherent perspective rooted in football knowledge (the post indicates at least some understanding of football transfer fees and contracts), but the assertion the poster is making is implied rather than formed into a coherent perspective.

> "With Romero and Dragusin leaving, we would have four, not really a crazy number."

This is borderline between Standard and Low-effort. The core of the post is pure opinion about the transfer fee, which would pull the comment towards Low-effort. However, it includes some reference to two players leaving and the depth-chart for that position, which would pull the comment towards Standard. I ultimately labeled the comment Standard, deciding that the post pulled more towards a common fan understanding about position depth.

## Fine-tuning Approach

Base model: `distilbert-base-uncased`
Training platform: `Google Colab (free GPU)` on a T4 GPU

**Training set distribution:**
Train: 142 examples
Validation: 31 examples
Test: 31 examples

**Train label distribution:**
low-effort    58
standard      56
informed      28

**Test label distribution:**
low-effort    13
standard      12
informed       6

### Hyperparameter Config

> Batch size: 4

The scarcity of informed posts and the need to ensure informed posts made up at least 20% of the example set meant that the overall size of the example set was small (204 examples). After split, the training set was only 142 examples. A batch size of 4 provides the model with more weight updates per pass through the training set. These more frequent updates allow the model to discern the more nuanced differences between labels, informed and standard in particular. Downside accepted: each batch is less representitive of the overall set.

> Learning rate: 3e-5

A slightly higher learning rate complements the increase in steps from the batch size adjustment.

> Epochs: 4

An additional epoch gives the model enough gradient updates to detect the subtle language differences rather than collapsing labels. More than 4 epochs and validation loss starts increasing indicating overfitting.

> Warmup_steps: 0.1

When set to a float, the value acts as a ratio of total training steps. Given the training set size, the model warms up for ~14 steps and then hits its learning rate for the remaining ~128 steps. With default settings (50 steps and 16 batch size), given the training set size, the model does not ever reach its full training rate (142 / 16 = ~9 steps per epoch x 4 epochs = ~36 steps total). 

## Evaluation

### Baseline Results

🎯 Baseline accuracy: 0.645  (evaluated on 31/31 parseable responses)

Per-class metrics (baseline):
              precision    recall  f1-score   support

    informed       0.50      1.00      0.67         6
    standard       0.67      0.33      0.44        12
  low-effort       0.77      0.77      0.77        13

    accuracy                           0.65        31
   macro avg       0.65      0.70      0.63        31
weighted avg       0.68      0.65      0.62        31

## Baseline Results

Model used: `llama-3.3-70b-versatile`

Prompt used:
``` 
You are classifying posts about player transfer rumors at Tottenham Hotsput
Football Club from the fan subreddit r/coys.
Assign each post to exactly one of the following categories.

informed: Contains a specific fact, detail, or context not already obvious to a
knowledgeable fan (e.g. injury history, prior transfer links, performance
elsewhere, club history, cross-fanbase sentiment, financial/regulatory context),
but not confident or well-reasoned opinion alone.
Example: "eh that was mostly Kwasniok waffling imo. He got most of his g+a in
starts (11 in 20 starts). After Kwasniok was sacked he started every game and
still got 3 goals and an assist in the last 7 games. His record of the bench
was impressive but I think thats just cause he is a real good player and will
get things done no matter what"

standard: A coherent football opinion built on narrative or belief rather than
verifiable evidence, which may reference real players, fees, or contracts,
but adds no fact a knowledgeable fan wouldn't already have, using known
information to support an opinion rather than inform.
Example: "I feel we should buy two attackers anyways though, Koln accepted a
roughly £40m bid for him from Brentford, if we add Savinho as well, we'll spend
maybe just under £100m for both."

low-effort: An emotional reaction, headline restatement, or vague/unsupported
assertion that could've been written by someone who only read the headline,
requiring no football knowledge, player familiarity, or analytical thought.
Example: "I'm sure they like money more than they hate us"

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
informed
standard
low-effort
```

## Evaluation Report

Epoch	Training Loss	Validation Loss	Accuracy
1	0.901827	0.796798	0.612903
2	0.472089	0.620465	0.774194
3	0.305678	0.483301	0.838710
4	0.139297	0.491978	0.806452

Per-class metrics (fine-tuned model):
              precision    recall  f1-score   support

    informed       1.00      0.67      0.80         6
    standard       0.62      0.83      0.71        12
  low-effort       0.82      0.69      0.75        13

    accuracy                           0.74        31
   macro avg       0.81      0.73      0.75        31
weighted avg       0.78      0.74      0.75        31

Wrong predictions: 8 / 31

--- #1 ---
Text:      Doesn't matter when Man Utd aren't willing to pay the 80 mil West Ham want. It worked with Mbeumo because he wasn't open to joining us and only wanted Man Utd. This is obviously different
True:      informed
Predicted: standard  (confidence: 0.82)

--- #2 ---
Text:      Where will that leave Robbo, Udogie and maybe even Spence then? Robbo hasn't gone to Spurs to be a backup again.
True:      low-effort
Predicted: standard  (confidence: 0.74)

--- #3 ---
Text:      You have to hit your quota of Brighton players this window. So if not him, your owners will make sure you bid for someone else.
True:      low-effort
Predicted: standard  (confidence: 0.68)

--- #4 ---
Text:      Had 19 G/A for Girona. Quite a lot of end product I'd say
True:      informed
Predicted: standard  (confidence: 0.53)

--- #5 ---
Text:      I'm hearing we will be announcing Tonali HWG tonight. I don't know how true it is but that's the buzz gaining steam on Social Media right now.
True:      low-effort
Predicted: standard  (confidence: 0.72)

--- #6 ---
Text:      This doesn't seem very likely. Though just the possibility should really make our resident young midfielders feel valued.
True:      low-effort
Predicted: standard  (confidence: 0.85)

--- #7 ---
Text:      I suspect it's more likely £40m + add ons
True:      standard
Predicted: low-effort  (confidence: 0.57)

--- #8 ---
Text:      Anderson not looking likely to move
True:      standard
Predicted: low-effort  (confidence: 0.65)

### Sample classifications

## Results Reflection

## Spec Reflection

**One way the spec helped**
I think by providing Claude with my hard edge cases and justification along with my label definitions when asking it to assist in labeling, the labeling results were quite good. Of the 164 random examples I pulled for standard and low-effort examples for the final data set, I only had to override ~15% of them. The fact that these were standard and low-effort instead of informed posts, the boundaries of which are more clearly defined, could be a contributing factor in its success.

**One way my implementation diverged**
My data collection plan was quite underbaked. After initially finding a number of informed posts on a single thread, I expected it to be easy to reach 1/3rd distribution across the set. However, after reviewing more threads it became clear just how rare informed posts were - only a handful per thread. As a consequence I had to pull these from 9 different threads. I wanted the other labels to be represented by this breadth as well, but had to ensure that informed posts were not underrepresented in the final data set. These reasons motivated the data collection approach detailed previously. 

## AI Usage 

**What I directed the AI to do:**

**What the AI did:**


**What I changed or overrode:**

**What I directed the AI to do:**

**What the AI did:**

**What I changed or overrode:**

