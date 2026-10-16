# Recommendation design and evaluation

[日本語](design.ja.md) · [Portfolio](README.md)

## Scope and provenance

The active implementation is Pearson user-based collaborative filtering, entered through `PearsonRecommendationService`. In 2026, Codex assisted with defect review, deterministic calculation extraction into `PearsonCourseRecommender`, tests and evaluation. Existing Mahout and KMeans experiments are retained in the private application but disabled by default; they are not validated alternatives.

## Data flow

1. Map login usernames to numeric student IDs and course codes to database IDs; ignore orphan records.
2. Exclude all enrolled courses. Only confirmed enrollments with finite ratings from 1 to 10 enter the rating matrix.
3. Calculate Pearson similarity over at least two commonly rated courses. Zero variance yields zero similarity; only positive finite similarities contribute.
4. Predict unseen ratings using `sum(similarity × neighbor rating) / sum(similarity)`. The prediction is not mean-centered and does not use negative neighbors.
5. Rank predictions by descending score and then course ID. Fill to five using unseen courses ordered by enrolled-student count and course ID.
6. Generate results in memory, then replace type-0 recommendation records in a transaction. Store predicted scores; zero marks a popularity fallback.
7. At query time, exclude current enrollments and duplicates again. Fill any remaining slots from available popular courses before pagination.

The private application's default schedule is every four minutes. The calculation engine has no Spring or database dependency. Course numeric IDs and business course codes are distinct; mapping them correctly was part of the repair.

## Evaluation protocol

The versioned fictional fixture contains **5 students, 31 courses and 26 ratings**. Each rating is held out once, giving 26 folds. The held-out enrollment and rating are removed from target similarity, exclusion and popularity counts. Relevance is the held-out observed course, not real-world click feedback.

| Metric | Pearson + fallback | Popularity baseline |
| --- | ---: | ---: |
| Precision@5 | 0.076923 | 0.076923 |
| Recall@5 / HitRate@5 | 0.384615 | 0.384615 |
| NDCG@5 | 0.277320 | 0.356225 |
| Personalized prediction coverage | 0.153846 (4/26) | N/A |
| MAE on personalized predictions | 1.5 (4 predictions) | N/A |

Precision counts hits divided by five per fold; recall records whether the single held-out course appears in the top five. NDCG discounts its rank. MAE excludes fallback items because they do not estimate ratings.

These results do not establish superiority over popularity: recall ties, NDCG is lower and personalized coverage is limited. They must not be presented as a resume accuracy gain.

## Recorded validation and limitations

The original project records 20 passing Maven tests and 27 passing HTTP acceptance checks on October 12, 2026. Tests cover similarity edge cases, deterministic order, ownership and recommendation behavior. This documentation update did not rerun the application suite. The standalone showcase does not contain the complete test harness or runnable application.

Limitations include tiny fictional data, no time-based split, no confidence intervals, no real-user outcome validation, no load benchmark and in-memory candidate scanning. Similarity shrinkage, overlap tuning and bias correction remain possible future work.
