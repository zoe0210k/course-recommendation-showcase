# 履修データの権限と整合性

[English](enrollment.md) · [プロジェクト概要](../README.ja.md)

出典：`CourseEnrollmentServiceImpl.java`。若依を基盤とする業務コードに対し、2026 年に Codex を活用して所有者チェックと重複登録処理を保守しました。抜粋内の例外メッセージは元の中国語を保持し、下表で訳を示します。

## Service 層の所有者チェック

画面上のボタンを隠すだけでは、他の学生のレコードへの直接リクエストを防げません。参照・評価・確定・削除の前に、認証済みユーザーと所有者を照合します。

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

既存の管理者ルールではアクセスを許可します。一般学生はログイン名と `studentId` の一致が必要です。新規登録も認証情報を使用し、一般学生による他人名義の登録を拒否します。

| 元のメッセージ | 日本語 |
| --- | --- |
| 缺少选课记录编号 | 履修レコード ID が必要です |
| 选课记录不存在 | 履修レコードが存在しません |
| 不能访问其他学生的选课记录 | 他の学生の履修情報にはアクセスできません |
| 已经选择该课程 | この科目は登録済みです |

## 同時リクエストによる重複への対応

存在確認の直後に INSERT する方式だけでは、同時リクエストで競合します。DB 側にも一意制約を設けます。

```sql
ALTER TABLE sys_select_course_info
ADD UNIQUE KEY uk_student_course(student_id, course_id);
```

上記は移行処理の主要な SQL を説明用に抜粋したものです。INSERT 時の一意制約違反を業務エラーに変換します。

```java
        try {
            return courseEnrollmentMapper.insertCourseEnrollment(courseEnrollment);
        } catch (org.springframework.dao.DuplicateKeyException e) {
            throw new ServiceException("已经选择该课程", 409);
        }
```

## 説明のポイント

- Service 層に置くことで、複数の入口で同じ所有者ルールを利用します。
- 一括削除では、すべての対象を検証してから削除します。
- 評価には履修確定が必要で、更新対象を評価・コメントに限定します。
- トランザクションは一意制約の代わりにはなりません。

抜粋はプロジェクトのエンティティ、Mapper、認証情報に依存し、単独ではコンパイルできません。[MIT ライセンス](../LICENSE)を参照してください。
