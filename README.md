# Knowledge-Based Recipe Recommender System
## Lee Littlejohn
#### June 6, 2020
---

## Overview

Now more than ever, people are eating at home. Home cooking can become monotonous, and sometimes food can spoil because we just don't know what to do with it. This system aims to provide inspiration and ideas for how to use up what you already have. Choice paralysis and apathy for cooking dinner _again_ can be detrimental to flavor and meal enjoyment. With the right suggestions, cooking dinner can be less wasteful and more interesting.

By collecting and processing a large number of recipes with the pandas, spaCy, and SciKit Learn libraries for Python, this project can take ingredients on hand as input and provide a three stage response.

1. The project displays a random sampling of recipes that include the ingredients on hand, displayed by difficulty quartile. If the user finds a recipe to be interesting, it can be selected for the next phase. If not, a new random sampling can be presented.

2. After vectorizing the filtered recipes from stage 1, recommendations based on the selected recipe are presented along with a visual representation of effort.

3. If the user does not have the requisite ingredients on hand for the recipe that most interests them from stage 2, suggestions can be presented for individual ingredient substitutions using a Gensim-like implementation of the spaCy library.

### Data Collection

The bulk of the data used comes from the _excellent_ [Recipe 1M+](http://pic2recipe.csail.mit.edu/) dataset, available for direct download [here](http://data.csail.mit.edu/im2recipe/recipe1M_layers.tar.gz). The images collected with this dataset have been discarded.

Other data that has been incorporated here includes the [Recipe Ingredients Dataset](https://www.kaggle.com/kaggle/recipe-ingredients-dataset) from Yummly presented by Kaggle and the [Recipe Box](https://eightportions.com/datasets/Recipes/#fn:1) dataset presented by Ryan Lee.

### Data Cleaning/EDA

These datasets were standardized with pandas to include columns for title, method, ingredients, and a corresponding url to find the original recipe (if it exists - only the Recipe 1m+ dataset had urls). This process is detailed in the data_imports.ipynb notebook in this repository.

In the ingredient_cleaning.ipynb also in this repository, the ingredients were isolated/tokenized. After dropping recipes with null titles, methods, and ingredients, and after dropping duplicates where both the method and title were duplicated (to preserve different recipes with the same title, like the numerous yet different instances of "guacamole"), the spaCy library was used to isolate ingredients.

With the exception of a few special cases, spaCy checked each listed ingredient for inclusion in the recommender phase by checking for root nouns or direct object nouns, like "prepared horseradish," where spaCy tags "horseradish" as a direct object. Special cases like "cloves of garlic" or "garlic cloves" where a "clove" is a unit of measure and not the true ingredient are detailed in the notebook.

The algorithm to isolate ingredients is expensive computationally and should only be used if the user has time to kill. It took approximately 4.5 hours to isolate the ingredients from this dataset. This process requires future optimization. After checking for the inclusion of some rare ingredients (asafoetida, kamut, etc.) nulls and duplicates generated by the ingredient isolation algorithm were also dropped. The final dataset with title, method, ingredients, url, and ingredient tokens can be accessed [here](https://www.dropbox.com/s/1x6b0jqw2eqoe21/compressed_clean.csv?dl=0). This file cannot be hosted on GitHub in the repository due to its size (392.86 MB).

### Difficulty/Effort Metrics

Home cooks frequently choose their meals based on the effort required to produce them, and that can mean many things. Effort can mean preparation time required, technical skills needed, the linguistic complexity of the recipe as it is written, or perhaps the sheer number of separate ingredients. Further investigation into these metrics is warranted and will likely be attempted continuously as the project is refined. The initial exploration of difficulty or effort metrics is in the difficulty_metrics.ipynb notebook.

To analyze the recipe methods for effort, three metrics were used. In a professional setting, cooks should always try to minimize action, that is, accomplish a task in the fewest moves possible. Therefore, the number of moves was extracted using spaCy. The "Moves" metric is a count of actions listed in the method of each recipe, such as "mix" or "add."

The technical difficulty of each move is calculated from a manually annotated list of verbs extracted from a random set of 60,000 sample recipes. Words like "spread" and "separate" are categorized into the lowest tier, whereas words like "quenelle" and "truss" are categorized into the highest tier. The full list of manually annotated verbs is included in thisrepository, concise_verbs.csv. These manual weights will likely be adjusted in the future. The combined total of adjusted weights will be referred to as the "L-Score" from here out.

For a measure of linguistic complexity, or "readability," the [LIX](https://en.wikipedia.org/wiki/Lix_(readability_test)) score was used, defined as:

$Readability = 100 * RLW + ASL$ where $RLW = \frac{n_{lw}}{n_w}$, $ASL = \frac{n_w}{n_s}$, $n_{lw} =$ number of words longer than 6 characters, $n_w =$ number of words, and $n_s =$ number of sentences.

A higher LIX score denotes higher difficulty, and a lower LIX score denotes ease. The LIX score is most useful when applied to longer documents. For instance, the method

"Combine the chicken, celery and green onions.
 Mix in salt and lemon juice; chill covered several hours.
 Add oranges, grapes and almonds.
 Toss with mayonnaise, being careful not to break up orange slices.
 Serve on lettuce leaves with crackers of your choice."
 
has an LIX score of 32, which is misleading, as this is a very easy recipe to read and understand, admittedly subjectively. To partially counteract this effect, the LIX score has been presented as the rounded 40% value of the LIX score, to better compare to the scale of the other metrics. The LIX $* .4$ of the example above is therefore 13. The LIX score was used in another study of recipe method difficulty as shown [here](https://dl.acm.org/doi/10.1145/3106668.3106673).

The five metrics used then are a count of ingredients ("Ngredients"), the L-Score, the LIX $* .4$, the number of moves ("Moves"), and an aggregate "Effort" score, defined as half of the sum of the other metrics.

### Recommender Structure

As described in the project overview, the first stage is to filter the entire dataset to jus those recipes that contain the ingredients the user has on hand. All effort metrics are applied to each recipe in this subset, and the results are presented by randomly sampling 4 recipes from each "effort" quartile.

By using an index of a presented recipe from the first stage, the top ten similar recipes to the recipe chosen are identified by cosine similarity after vectorizing the ingredients with SciKit Learn's Term-Frequency Inverse-Document-Frequency Vectorizer. Those recipes are then presented in a readable format along with a polar chart visually showing their assosciated effort metrics.

Very rough substitutions can be then be generated. These substitutions are from the underlying spaCy model which is _not_ focused on edible ingredients, and are thus most often unusable. However, I have found the substitutions to be interesting, generating potentially inventive new dishes.

### Next Steps and Shortcomings

Being a proof of concept project, there is much room for further improvement. Notably, then ingredient isolating algorithm does not support the extraction of unique ingredients with compound descriptors, like "green onions" or "bay leaves." The spaCy library supports model retraining, which will be attempted in the future to optimize both the ingredient isolation phase and ingredient substitution phase, which will undoubtedly improve performance during the recommender phase. The special cases of garlic cloves, herb sprigs, and hearts of romaine, celery, etc. are handled currently, but many more exist, like bay leaves. Also, although the units of measure are removed before ingredient extraction, those units remain if they are immediately followed by a period.

The effort metrics are very subjective, being a direct consequence of how I feel about certain techniques and how they fit in to recipes for the average person. All of these will be tuned in the future.

The next steps towards sharing this project with the average user consist of converting the dataset to a database and hosting it in the cloud, while converting the algorithms to a deployable app.


## References

[Recipe 1M+](http://pic2recipe.csail.mit.edu/)

[Recipe Ingredients Dataset](https://www.kaggle.com/kaggle/recipe-ingredients-dataset)

[Recipe Box](https://eightportions.com/datasets/Recipes/#fn:1)

[LIX Score](https://en.wikipedia.org/wiki/Lix_(readability_test))

[Calculating Cooking Recipe's Difficulty based on Cooking Activities](https://dl.acm.org/doi/10.1145/3106668.3106673)

[On the predictability of the popularity of online recipes](https://link.springer.com/article/10.1140/epjds/s13688-018-0149-5)

[RecipeNet](https://dominikschmidt.xyz/recipe-net/)

[What’s Cookin’? Interpreting Cooking Videos using Text, Speech and Vision](https://www.aclweb.org/anthology/N15-1015.pdf)

Rounak Banik, "Hands-On Recommendation Systems with Python" 2020 Oreilly Media


## Most Influential and Divisive Words 'salt', 'sugar', 'oil', 'pepper', 'butter', 'flour', 'garlic', 'onion', 'cheese', 'water'