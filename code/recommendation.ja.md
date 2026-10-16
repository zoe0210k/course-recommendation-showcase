# 推薦順位と人気科目による補完

[English](recommendation.md) · [プロジェクト概要](../README.ja.md)

出典：`PearsonCourseRecommender.java` の `recommend` メソッド末尾。2026 年に Codex を活用して計算処理を分離・保守しました。

## 課題と対応

個別予測だけでは候補が不足する場合があります。同点時の順序と、人気科目による補完時の重複にも対応します。この処理に入る前に、`available` から履修済み科目を除外し、`predicted` に正の類似度による加重予測、`popularity` に履修学生数を格納しています。

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

予測候補をスコア降順・科目 ID 順で優先します。不足分は人気順・科目 ID 順で追加し、`chosen` で重複を防ぎます。補完項目のスコア 0 は補完の識別値であり、学生の評価点ではありません。候補数が少なければ `limit` 未満の結果を返します。

## 説明のポイント

- 同点時の順序を固定し、再現性とテストの安定性を確保します。
- 定期処理後に履修登録された科目も、取得時に再度除外します。
- 人気科目による補完は候補を確保する仕組みで、精度向上の根拠ではありません。
- 候補をメモリ上で走査しており、大規模な性能検証は行っていません。

抜粋のみではコンパイルできません。[MIT ライセンス](../LICENSE)を参照してください。
