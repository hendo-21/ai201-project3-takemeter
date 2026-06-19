# TakeMeter
TakeMeter is a fine-tuned text classifier that evaluates discourse quality regarding player transfer discussions within r/coys, the Tottenham Hotspur subreddit.


## Community
<!-- What community did you choose and why? Why is this community a good fit for a classification task — what makes the discourse varied enough to be interesting? -->
As a lifelong fan of Tottenham Hotspur Football Club, I am deeply familiar with the online discourse surrounding player transfer news, which led me to choose the club's subreddit, r/coys, for this project. Sorting the wheat from the chaff to find truly insightful perspectives on potential incoming transfers is often challenging at a glance. What makes this rumor mill fun to follow is uncovering those nuggets of genuine analytical commentary that spark excitement about a potential signing.

Crucially, discourse quality in this community leaves detectable signals in the text itself. Insightful posts tend to contain named entities, quantitative claims, tactical vocabulary, and evidential language. Because these distinct textual markers separate high-quality analysis from low-effort speculation, they provide excellent features for a text classifier to learn and detect.

## Labels
<!-- What are your 2–4 labels? Define each in a complete sentence. Include 2 example posts per label. -->

### Insightful
A post that tells the reader something they didn't already know or assume, using specific, verifiable evidence — player statistics, transfer market data, first-hand match observation, or external context not in the headline.

**Key signal:** The post contains a specific, checkable piece of evidence about the player or deal (not just a generic stat confirming an existing assumption), and that evidence changes or adds to what an informed fan already believed. If the post states a fact and then appends a generic reaction or truism without doing anything further with it, it belongs in Standard.

**Examples**
> "eh that was mostly Kwasniok waffling imo. He got most of his g+a in starts (11 in 20 starts). After Kwasniok was sacked he started every game and still got 3 goals and an assist in the last 7 games. His record of the bench was impressive but I think thats just cause he is a real good player and will get things done no matter what"

> "He was on a tear last season. He single-handedly saved Koln from relegation. Just watch highlights from their final game against Bayern for his goal to get a sense of his skill set. There was quite a stink that Nagelsmann didn't select him for the national team and with Sane instead."

### Standard
A post that contributes a coherent perspective rooted in football knowledge or fan context, but whose claims rest on narrative and belief rather than verifiable evidence or specific data.

**Key signal:** The post may appear specific, referencing real players, fees, or contract structures, but its claims are either speculative, hypothetical, or constructed from fan narrative rather than verifiable information. Data points, if present, are scaffolding for opinion rather than evidence for a conclusion.

**Examples**

> "I feel we should buy two attackers anyways though, Koln accepted a roughly £40m bid for him from Brentford, if we add Savinho as well, we'll spend maybe just under £100m for both."

> "Loaning Vuskovic back to Hamburg is probably the ideal situation. Let him fulfill his dream playing with his brother and he will be grateful with TOT. Before he left make him sign a new 5-year deal with bumper wages and let Hamburg covers the first year. Assuming Romero and Drag left this window and Danso next then we will have him competing with VDV, PVH and Senesi hopefully for a CL challenging squad in 2028."

### Low-effort
A post that contributes nothing beyond an emotional reaction, headline restatement, or assertion so vague or unsupported that it adds no value to the discussion.

**Key signal:** The post could have been written by someone who only read the headline. It requires no football knowledge, no familiarity with the player, and no analytical thought to produce.

**Examples**
> "Honestly a great signing and after the news that was coming out it’s cheaper than I was expecting"

> "Maybe Jan & Micky can be the new Jan & Toby"


## Hard Edge Cases
<!-- What type of post will be genuinely ambiguous between two labels? How will you handle it when you encounter it during annotation? -->

> "El Mala is genuinely superb. Isn't there an issue that his family want his brother to move with him though?"
This is borderline between Standard and Low-effort primarily because of the second sentence. The first sentence contributes a perspective with implied knowledge about the player. The second sentence also implies some level of knowledge that could genuinely be useful for the community. However, both sentences are too vague and lack any supporting evidence to be useful; the poster is asking someone else to verify their claim in the second sentence.

> "52m for a guy with 1 year left on his contract is mad. But I will take it, what ever De Zerbi wants"

This is borderline between Standard and Low-effort because of the coherent perspective rooted in football knowledge (the post indicates at least some understanding of football transfer fees and contracts), but the assertion the poster is making is implied rather than justified.

> "Do you actually know anything about defending in football? Or just a little Fifa and FM? It is as much about reading the game and savvy as it is about brute physicality - skills that take time to hone. By even the most optimistic accounts, he's still got a ways to go defending-wise and is slow on the turn. The bundesliga is a good league, but the premier league is that much faster, chocked full of elite attackers he'll have to face every week. In the Bundesliga he'd play against them maybe ten times a year. Ask yourself: How many 19 year old CBs have you seen even play in the PL? How many of them have contributed to teams competing for Europe?"

This is borderline between Insightful and Standard because it provides grounded reasoning in league knowledge (references to physicality) and knowledge of the player (reference to his limitations). However the conclusion the poster makes is not particularly novel (Premier leauge is widely considered the top league in Europe), and the evidence is rather subjective.

## Data Collection Plan
<!-- Where will you collect examples? How many per label? What will you do if a label is underrepresented after 200 examples? -->
I will collect examples from the subreddit r/coys, selecting posts regarding transfer news (rumors and confirmed transfers). For the Insightful label, I will sort the post by Top, and then manually select posts for that label beginning with top post trees. The target distribution for each label is 1/3rd per label, so I will aim for ~67 examples per label. Insightful posts are the minority of posts on the subreddit, so they will be overrepresented in the training dataset relative to their distribution in overall reddit post history, however this intentional oversampling ensures sufficient Insightful representation for training. I will collect Standard and Low-signal posts by scraping transfer-related threads using [tool], filtering for comments that are direct replies to transfer news headlines. Non-transfer posts encountered during collection will be excluded. If Standard or Low-effort posts are underrepresented after 200 examples, I will manually collect examples. This may result in an overrepresentation of that label relative to the dataset, as discussed previously, but will ensure sufficient representation for that label for training.

## Evaluation Metrics
<!-- Which metrics will you use to evaluate your model and why are those the right ones for this specific task? (Accuracy alone is not enough — explain what else you need and why.) -->

**Overall accuracy for trained and baseline models.**
This metric is useful for determining if the model training actually improved its ability to classify the text. However, the labels represent three completely different types of discourse and the model could perform well on classifying posts for 2/3 classes, but miss one entirely and still generate a reasonably high accuracy score. The Insightful/Standard boundary is inherently more subjective than the Standard/Low-signal boundary, making it likely that the model will struggle to distinguish these two classes specifically.

**Per-class metrics**
Per-class metrics are necessary to determine whether the model is learning all three categories independently or collapsing distinctions between harder-to-separate classes. I'll be using three: precision, recall, and F1. Precision measures how much you can trust the model when it makes a prediction. A model that fails to surface the most insightful posts likely has little utility for someone developing a community tool. Recall measures how much the model misses. Similarly, a model that misses the most insightful posts similarly likely lacks utility. F1 is a measure of how balanced the model is on precision and recall, and allows for ease of comparison with the baseline model. A model that is strong on precision, but poor on recall would show up in this metric, and likely carries different utility for someone developing a community tool or using it to conduct research.

**Confusion matrix**
The confusion matrix will surface which classes the model confused with which. This will reveal whether the errors are clustered around the boundaries, or the model is making more critical errors: like classifying Low-effort posts as Insightful.

## Definition of Success
<!-- What performance would make this classifier genuinely useful? What would you accept as "good enough" for deployment in a real community tool? -->
The community tool I envision is a browser extension that surfaces insightful posts for a user for a given Reddit thread on player transfer news. This requires that the model is precise enough that the user can trust it AND the model surfaces enough insightful posts that the tool actually has utility. A F1 score > 0.65 would indicate that the model is not surfacing too many Low-effort posts as Insightful, and is not missing too many Insightful posts that would be useful to the user.

## AI Tool Plan

**Label Stress-testing**
I will give Claude my list of definitions and ask it to generate 5 posts that sit at the border of each label. I will then see if I can cleanly label the generated posts, and if not, I will refine my definitions and ask it to generate 3 more posts for review.

**Annotation assistance**
Since I am manually gathering Insightful posts, I will use LLM assistance for labeling Standard and Low-effort posts. I will give Claude (`claude-sonnet-4.6`)my finalized label definitions and key signals and ask it to label posts in batches of 25. After each batch, I will review every label and override where necessary.

The output will be a CSV tracking both the LLM's original label and my final label, so I can compute an agreement rate between my judgment and the LLM's pre-labels. Manually-sourced Insightful posts will not have an LLM label, since they bypass the pre-labeling step entirely. Output format:

| Column            | Type      | Description                                                                    |
|--------------------|-----------|-------------------------------------------------------------------------------|
| thread_title       | string    | Submission headline (for reference/context during labeling, not model input)  |
| comment_text       | string    | The comment body — this is the model's actual input                           |
| llm_label          | string    | Label assigned by Claude during pre-labeling pass (null if not LLM-assisted)  |
| final_label        | string    | Your final label — what actually goes into training                           |
| llm_assisted       | bool      | True if this example went through LLM pre-labeling at all                     |
| overridden         | bool      | True if final_label differs from llm_label                                    |
| notes              | string    | Edge case reasoning, uncertainty flags, "edge: insightful/standard" tags etc. |
| split              | string    | train / validation / test (assigned after collection, before training)        |

**Failure analysis**
I will give Claude my list of wrong predictions and ask it to identify patterns in the posts that could point to possible failure causes with the model. I will provide it my label definitions for reference.