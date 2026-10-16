# Ranking and popularity fallback

[日本語](recommendation.ja.md) · [Portfolio](../README.md)

Source: the end of `recommend` in `PearsonCourseRecommender.java`. Extraction and maintenance were assisted by Codex in 2026.

## Problem and approach

Personalized predictions can be sparse, scores can tie, and popularity fallback must not repeat predicted courses. Before this excerpt, `available` excludes enrollments, `predicted` holds weighted predictions from positively similar neighbors, and `popularity` counts enrolled students.

```java
        predicted.sort(Comparator.comparingDouble((Choice c) -> c.score).reversed().thenComparingLong(c -> c.courseId));
        List<Choice> output = new ArrayList<>(); Set<Long> chosen = new HashSet<>();
        for (Choice choice : predicted) { if (output.size() == limit) break; output.add(choice); chosen.add(choice.courseId); }
        List<Long> fallback = new ArrayList<>(available);
        fallback.sort(Comparator.comparingInt((Long id) -> popularity.getOrDefault(id, 0)).reversed().thenComparingLong(Long::longValue));
        for (Long course : fallback) {
            if (output.size() == limit) break;
            if (chosen.add(course)) output.add(new Choice(course, 0, false));
        }
        return output;
```

Predicted items come first, ordered by descending score and then course ID. Fallback items are ordered by popularity and then course ID. `chosen` prevents duplicates. A fallback score of zero marks its origin; it is not a student's rating. Fewer available courses may yield fewer than `limit` results.

## Discussion points

- A stable tie-breaker makes results reproducible.
- Query-time filtering excludes courses enrolled after the scheduled recommendation run.
- Fallback improves availability; it does not establish an accuracy gain.
- Candidate scanning is in memory; large-scale performance has not been validated.

This excerpt cannot compile by itself. See the [MIT license](../LICENSE).
