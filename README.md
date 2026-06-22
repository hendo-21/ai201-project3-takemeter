# TakeMeter
TakeMeter is a a fine-tuned text classifier that evaluates player transfer discourse quality in the Tottenham Hotspur Football Club fan subreddint r/coys.

---

## Community
As a lifelong fan of Tottenham Hotspur Football Club, I am deeply familiar with the online discourse surrounding player transfer news, which led me to choose the club's subreddit, r/coys, for this project. Sorting the wheat from the chaff to find truly informed perspectives on potential incoming transfers is often a challenge requiring constant scrolling to find the few nuggets of genuine insight. Discourse quality in this community leaves detectable signals in the text itself. Informed posts tend to contain named entities (people, clubs, leagues), quantitative claims (stat comparisons), tactical vocabulary (positions and formations), and evidential language (specific first-hand viewing experiences). Because these distinct textual markers separate high-quality analysis from standard fan-narrative based speculation or low-effort reaction, they provide features for a text classifier to learn and detect. A model fine-tuned to classify this discourse could be used to develop a tool that surfaces only the most informed takes for a fans looking to get a quick read on the potential transfer.

---

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

---

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

---

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

---

## Zero-shot baseline

To establish a zero-shot baseline, Groq's `llama-3.3-70b-versatile` was tasked with using provided label definitions to label the 31 examples from the test set. The prompt used is shared below:

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

---

## Evaluation Report

### Overall accuracy

==================================================
RESULTS COMPARISON
==================================================
Model                               Accuracy
---------------------------------------------
Zero-shot baseline (Groq)              0.677
Fine-tuned DistilBERT                  0.742
---------------------------------------------

### Per model per-class metrics

**🎯 Baseline accuracy: 0.677  (evaluated on 31/31 parseable responses)**

**Per-class metrics (baseline):**
              precision    recall  f1-score   support

    informed       0.60      1.00      0.75         6
    standard       0.62      0.42      0.50        12
  low-effort       0.77      0.77      0.77        13

    accuracy                           0.68        31
   macro avg       0.66      0.73      0.67        31
weighted avg       0.68      0.68      0.66        31

**🎯 Fine-tuned model accuracy: 0.742**

**Per-class metrics (fine-tuned model):**
              precision    recall  f1-score   support

    informed       1.00      0.67      0.80         6
    standard       0.62      0.83      0.71        12
  low-effort       0.82      0.69      0.75        13

    accuracy                           0.74        31
   macro avg       0.81      0.73      0.75        31
weighted avg       0.78      0.74      0.75        31

Epoch	Training Loss	Validation Loss	Accuracy
1	0.901827	0.796798	0.612903
2	0.472089	0.620465	0.774194
3	0.305678	0.483301	0.838710
4	0.139297	0.491978	0.806452

### Confusion Matrix

| | Predicted: Informed | Predicted: Standard | Predicted: Low-effort |
|---|---|---|---|
| **True: Informed** | 4 | 2 | 0 |
| **True: Standard** | 0 | 10 | 2 |
| **True: Low-effort** | 0 | 4 | 9 |

### Wrong Prediction Analysis

Wrong predictions: 8 / 31. Four shared and analysed below.

--- #1 ---
Text:      Doesn't matter when Man Utd aren't willing to pay the 80 mil West Ham want. It worked with Mbeumo because he wasn't open to joining us and only wanted Man Utd. This is obviously different
True:      informed
Predicted: standard  (confidence: 0.82)

This post contains contextually rich insights in the form of transfer fees, cross-club references, and names a player, but the model predicted this was standard rather than informed. It also did so with very high confidence. I believe the model was interpreting the the overall sentiment as fan narrative rather than genuinely new transfer saga context. When reading the thread it is clear that this is genuinely new information with a varifiable assertion: Mbuemo only wanted Man Utd. (well reported fact), but with the thread context lacking, the model interpreted the language as speculative. It is possible the model sees emphatic language like "obviously" and "only wanted..." as typical of standard posts, which are based on fan narrative or belief rather than verifiable evidence.

--- #3 ---
Text:      You have to hit your quota of Brighton players this window. So if not him, your owners will make sure you bid for someone else.
True:      low-effort
Predicted: standard  (confidence: 0.68)

Reading this post within the context of the thread, one quickly sees that this is an emotional and sarcastic response to Spurs being linked to another player from Brighton. It is purely an emotional reaction from a rival fan, and the assertion is entirely unsupported which makes low-effort a more appropriate label. However, without that fan and thread context, the model incorrectly predicts it as standard. It is possible that the model learned that standard posts typically contain conditional or logical structures ("you have to hit your quota... so if not him...") that make those posts sound like a coherent perspective.  

--- #4 ---
Text:      Had 19 G/A for Girona. Quite a lot of end product I'd say
True:      informed
Predicted: standard  (confidence: 0.53)

This post provides genuinely new information to the discussion (the stat about the goals and assists the player had for a previous club), which is the definition of informed. Given the lower confidence, it is possible the model was conflicted between the specificity of the stat, and the second part of the sentence which uses more speculative language. Looking at the dataset, informed posts were generally 2-3x as long as standard or low-effort posts, so it's possible the model also learned that informed posts are typically much longer and made a prediction based on post length.

--- #7 ---
Text:      I suspect it's more likely £40m + add ons
True:      standard
Predicted: low-effort  (confidence: 0.57)

This is a coherent perspective based on fan understanding of typical transfer fees for this type of player. It adds some value to the discussion and appears like it could be in response to the headline which contains a transfer fee. These two components edge the example closer to standard than low-effort, but the model predicts (with relatively low confidence) that it is low-effort. I suspect the post length is the primary driver for the model's prediction of low effort given the content contains a speculated transfer fee that is likely in response to the rumored fee, thus required some analytical thought.

### Sample classifications

**Prediction with analysis:**
> "Wharton's out of possession is something you need to build around. Ground duels won: Anderson 2.06 p90 (81st) vs Wharton 1.06 p90 (27th). win rate 36.9% (74th) vs 31.2% (20th). Defensive aerials won. Anderson 0.96 p90 (84th) vs Wharton 0.27 p90 (29th). win rate 56.0% (75th) vs 45.4% (11th). Aerial duels won overall. Anderson 1.70 p90 (84th) at 55.2% (86th). Wharton 0.84 p90 (47th) at 46.8% (31st)"
Label: informed     Confidence: 0.65

This a textbook example of an informed post: it makes a claim and then supports it with new and specific information in the form of comparative statistics between two players of a similar position. The most surprising element of this example is that the model was not more confident. It's possible the language used in the first sentence ("you need to...") is more typical of standard posts.

**Additional samples**

> Madders played at 110% and it really showed in injuries & muscle soreness. If he focused on his best qualities, passing and decisiveness, he could stay healthier. Easier said than done among professional athletes.
Label: standard     Confidence: 0.64

> He picked up nine yellow cards this season, same as Van de Ven and Romero
Label: standard     Confidence: 0.54

> Honestly a great signing
Label: low-effort   Confidence: 0.92

---

## Results Reflection

Overall, the model performed better than the baseline and better than my target F1 score (0.75 actual vs 0.65 target) I identified in my `planning.md` file. Therefore, I believe it could actually be used to create some sort of community tool to surface informed posts for users (I picture a browser extension of some kind) and provide some value. The standard / low-effort boundary was hardest for the model to parse, as seen in the confusion matrix. This seems in part due to labeling issues. Consider the following examples:
```
--- #2 ---
Text:      Where will that leave Robbo, Udogie and maybe even Spence then? Robbo hasn't gone to Spurs to be a backup again.
True:      low-effort
Predicted: standard  (confidence: 0.74)

--- #5 ---
Text:      I'm hearing we will be announcing Tonali HWG tonight. I don't know how true it is but that's the buzz gaining steam on Social Media right now.
True:      low-effort
Predicted: standard  (confidence: 0.72)
```

Example #2 is not a post that could be made without familiarity with the players or general football knowledge. That fact alone pushes this example more towards standard than low-effort, so I'd say the model got it right. Example #5 provides genuinely new information to the discussion, the defining characteristic of informed, even if the post later uses hedging language. These two examples reveal the challenges of correctly labeling large datasets. Labelers make mistakes or change their perspective over labeling sessions and that can impact what the model learns. To correct this, I would take another pass at reviewing the data and see if the model performance improves.

Additionally, as speculated at above, the model seems to have learned that posts containing concessive ("although", "though", "despite") and conditional ("if", "unless") language are good indicators of a coherent perspective typical of standard posts. Consider the following examples for why this is an imperfect approach:
```
--- #3 ---
Text:      You have to hit your quota of Brighton players this window. So if not him, your owners will make sure you bid for someone else.
True:      low-effort
Predicted: standard  (confidence: 0.68)

--- #6 ---
Text:      This doesn't seem very likely. Though just the possibility should really make our resident young midfielders feel valued.
True:      low-effort
Predicted: standard  (confidence: 0.85)
```
Example #3 is a banter dressed up in conditional structure with no real football claim underneath it. A knowledgable fan would recognize that immediately. The concessive part of example #6 is vague sentiment with no specific content. I suspect that if there were more low-effort examples in the dataset that had a similar structure the model would have struggled.

As noted above, informed comments were siginficantly longer than standard and low-effort posts (~100 words vs ~30 for standard and ~12 for Low-effort), and given the failure on short informed posts, it is likely the model learned that post length is a good indicator of informed posts. The higher post length for informed posts suggests that during annotation I may have unconsciously required more elaboration from Informed comments than the definition strictly demands. It's possible that if I had let the LLM label informed posts in a thread as well, I would have had a list of examples that was more diverse in length. Tightening annotation consistency on short informed cases would improve label quality.

As noted above, informed comments were significantly longer than standard and low-effort posts (~100 words vs ~30 for Standard and ~12 for Low-effort). Given the model's failures on short informed posts (e.g. "Had 19 G/A for Girona" — predicted Standard) and its tendency to predict standard for longer low-effort posts with conditional or causal sentence structure it's possible the model learned post length as a proxy for class rather than the deeper content distinction the definitions require. The higher word count for informed posts suggests that during annotation I may have unconsciously required more elaboration from informed comments than the definition strictly demands. Tightening annotation consistency on short factual comments would likely improve label quality in future passes.

## Spec Reflection

**One way the spec helped**
I think by providing Claude with my hard edge cases and justification along with my label definitions when asking it to assist in labeling, the labeling results were quite good. Of the 164 random examples I pulled for standard and low-effort examples for the final data set, I only had to override ~15% of them. The fact that these were standard and low-effort instead of informed posts, the boundaries of which are more clearly defined, could be a contributing factor in its success.

**One way my implementation diverged**
My data collection plan was quite underbaked. After initially finding a number of informed posts on a single thread, I expected it to be easy to reach 1/3rd distribution across the set. However, after reviewing more threads it became clear just how rare informed posts were - only a handful per thread. As a consequence I had to pull these from 9 different threads. I wanted the other labels to be represented by this breadth as well, but had to ensure that informed posts were not underrepresented in the final data set. These reasons motivated the data collection approach detailed previously. 

---

## AI Usage 

**What I directed the AI to do:**
Label all Reddit comments using the Informed / Standard / Low-effort classification scheme, following a detailed specification including label definitions, output format, and instructions to skip bot/deleted comments and flatten nested replies.

**What the AI did:**
Produced a CSV with labels placed in `llm_label` column and all required metadata fields. 

**What I changed or overrode:**
Randomly sourced ~80 standard and low effort examples from the generated CSV and then reviewed all labeled examples manually, correcting cases where the model misapplied labels (primarily at the boundaries of standard / informed and standard / low-effort). Stored final label decisions in the `final_label` column.

**What I directed the AI to do:**
Debug a 100% unparseable baseline result in the fine-tuning Colab notebook.

**What the AI did:**
Identified the unparseable baseline as a case-sensitivity bug in the notebook's label matching logic and rewrote the classifier function. 

**What I changed or overrode:**
Investigated the case-sensitivity fix independently rather than accepting the function rewrite and resolved the error by lowercasing the labels in the source CSV instead.

---
