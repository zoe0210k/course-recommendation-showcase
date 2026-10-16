# Enrollment authorization and consistency

[日本語](enrollment.ja.md) · [Portfolio](../README.md)

Source: `CourseEnrollmentServiceImpl.java`. The business code is based on RuoYi; ownership checks and duplicate handling were maintained with Codex assistance in 2026. Source excerpts retain the original Chinese exception messages; translations follow below.

## Ownership at the Service layer

Hiding a UI button cannot stop a direct request for another student's record. Reads, ratings, confirmations and deletions check the authenticated user's ownership:

```java
    private CourseEnrollment requireOwned(Long id) {
        if (id == null) throw new ServiceException("缺少选课记录编号", 400);
        CourseEnrollment record = courseEnrollmentMapper.selectCourseEnrollmentById(id);
        if (record == null) throw new ServiceException("选课记录不存在", 404);
        if (!SecurityUtils.isAdmin(SecurityUtils.getUserId())
                && !Objects.equals(record.getStudentId(), SecurityUtils.getUsername())) {
            throw new AccessDeniedException("不能访问其他学生的选课记录");
        }
        return record;
    }
```

The existing administrator rule permits access; a regular student's login username must match `studentId`. Enrollment creation also derives identity from authentication and rejects attempts by regular students to enroll for someone else.

| Original message | Meaning |
| --- | --- |
| 缺少选课记录编号 | Enrollment record ID is required |
| 选课记录不存在 | Enrollment record not found |
| 不能访问其他学生的选课记录 | Access to another student's enrollment is denied |
| 已经选择该课程 | Already enrolled in this course |

## Concurrent duplicate requests

A check followed by an insert can race. The project additionally uses a database uniqueness constraint:

```sql
ALTER TABLE sys_select_course_info
ADD UNIQUE KEY uk_student_course(student_id, course_id);
```

This is the core statement from the migration, shown for review only. An insert catches uniqueness violations and returns a business error:

```java
        try {
            return courseEnrollmentMapper.insertCourseEnrollment(courseEnrollment);
        } catch (org.springframework.dao.DuplicateKeyException e) {
            throw new ServiceException("已经选择该课程", 409);
        }
```

## Discussion points

- Service-layer checks share the ownership rule across entry points.
- Batch deletion validates every record before deleting.
- Ratings require confirmed enrollment and update only rating/comment fields.
- A transaction does not replace a uniqueness constraint.

The excerpts depend on project entities, mappers and security context; they cannot compile alone. See the [MIT license](../LICENSE).
