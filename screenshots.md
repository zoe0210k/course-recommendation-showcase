# Screenshot reading guide

[日本語](screenshots.ja.md) · [Portfolio](README.md)

The application currently uses a Chinese UI. These are existing demonstration screenshots, not translated UI mockups. The vocabulary below explains the course workflow; it is not a claim that every label appears in every image.

## Course browsing

![Course browsing](images/courses.png)

Students browse course information and select a course for pre-enrollment. The server obtains student identity from authentication rather than trusting a submitted student ID.

## Enrollment and rating

![Enrollment and rating](images/enrollments.png)

The workflow distinguishes pre-enrollment (state 1) and confirmed enrollment (state 2). Only confirmed enrollment can be rated. A rating is between 1 and 10; comments are limited to 400 characters.

## Recommendations

![Recommendations](images/recommendations.png)

The system recommends unseen courses using Pearson collaborative filtering and popularity fallback. The query excludes current enrollments again, so enrollment changes can affect the displayed list before the next scheduled calculation.

## UI vocabulary

| Chinese | English |
| --- | --- |
| 课程 / 课程名称 | Course / Course name |
| 课程编号 / 学分 | Course code / Credits |
| 查询 / 重置 | Search / Reset |
| 选课 / 预选 | Enroll / Pre-enroll |
| 确认选课 / 已确认 | Confirm enrollment / Confirmed |
| 退课 | Withdraw from a course |
| 评分 / 评论 | Rating / Comment |
| 推荐课程 / 热门课程 | Recommended courses / Popular courses |

## Suggested interview walkthrough

1. Explain the selected course and pre-enrollment state.
2. Confirm enrollment, then submit a rating and comment.
3. Show that current enrollments are excluded from recommendations.
4. Use the selected code excerpts to explain ownership checks and stable ranking.

This is a suggested demonstration sequence, not a claim that a new demo or test was executed for this documentation update.
