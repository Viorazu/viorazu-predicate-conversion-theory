# Conversion Patterns: 述語化変換の実装パターン集

## 概要

このドキュメントは、Viorazu.述語化変換理論の実装に必要な具体的な変換パターンを体系的にまとめたものです。AI翻訳システムやテキスト処理ツールでの実装時に参照してください。

---

## 1. 動詞グループ別変換パターン

### 1.1 品質・性能向上系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(improve\|enhance\|boost)` | efficiency | V+N | 効率を上げる | 処理効率を高める | |
| `(improve\|enhance\|boost)` | accuracy | V+N | 正確さを高める | 精度を上げる | |
| `(improve\|enhance\|boost)` | reliability | V+N | 信頼性を高める | 信頼性を高める | |
| `(improve\|enhance\|boost)` | performance | V+N | 性能を高める | 応答速度を上げる | 速度/スループットで分ける |
| `(increase\|raise)` | throughput | V+N | 処理量を増やす | スループットを上げる | |
| `(decrease\|reduce\|cut)` | latency | V+N | 遅延を減らす | レイテンシを下げる | |
| `(reduce\|cut)` | cost | V+N | 費用を減らす | コストを削減する | |

### 1.2 保証・確保系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(ensure)` | compatibility | V+N | 互換性を確保する | 互換性を確保する | 他でも動くようにする |
| `(ensure)` | privacy | V+N | プライバシーを守る | プライバシーを確保する | |
| `(ensure)` | security | V+N | 安全を守る | セキュリティを確保する | |
| `(maintain\|keep\|retain)` | consistency | V+N | 一貫性を保つ | 表記を統一する / データ整合性を保つ | |
| `(maintain\|keep\|retain)` | availability | V+N | 止めない | 可用性を保つ | 一般は平易に |

### 1.3 実現・達成系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(achieve\|attain\|realize)` | sustainability | V+N | 持続可能にする | 持続可能な運用にする | |
| `(achieve\|attain\|realize)` | compliance | V+N | 規則に従う | 準拠させる／遵守する | |
| `(enable\|allow\|let) ([^ ]+) to ([^ ]+)` | - | X to V | `\1`が`\2`できるようにする | `\1`が`\2`できるようにする | 正規表現で主語と動詞を拾う |
| `allow for` | customization | phrase | カスタマイズを可能にする | カスタマイズを可能にする | |
| `(enable)` | offline use | phrase | オフラインでも使えるようにする | オフライン動作を可能にする | |
| `(enable)` | integration | V+N | 連携できるようにする | 外部連携を可能にする | |

### 1.4 対処・処理系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(address\|tackle\|deal with)` | risk | V+N | リスクに対処する | リスクに対応する | |
| `(mitigate\|alleviate)` | risk | V+N | リスクを抑える | リスクを低減する | |
| `(optimize)` | functionality | V+N | 機能を最適化する | 機能構成を最適化する | 不要機能は削る |
| `(optimize)` | cost | V+N | 費用を最適化する | コストを最適化する | |
| `(optimize)` | memory usage | V+N | メモリの使い方を最適化する | メモリ使用量を最適化する | |

### 1.5 実装・導入系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(implement)` | feature | V+N | 機能を実装する | 機能を実装する | |
| `(implement)` | policy | V+N | 方針を導入する | ポリシーを導入する | |
| `(deploy)` | model | V+N | モデルを導入する | モデルをデプロイする | |
| `(support)` | feature | V+N | 機能に対応する | 機能をサポートする | |
| `(disable)` | feature | V+N | 機能を無効にする | 機能を無効化する | |
| `(activate\|enable)` | feature | V+N | 機能を有効にする | 機能を有効化する | |

### 1.6 検証・監視系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(verify\|confirm)` | result | V+N | 結果を確認する | 結果を確認する | |
| `(validate)` | hypothesis | V+N | 仮説を検証する | 仮説を検証する | |
| `(validate)` | input | V+N | 入力を確かめる | 入力を検証する | |
| `(monitor)` | performance | V+N | 性能を見張る | 性能を監視する | |
| `(monitor)` | drift | V+N | ずれを見張る | ドリフトを監視する | |
| `(track)` | metrics | V+N | 指標を追う | 指標を追跡する | |
| `(detect)` | anomaly | V+N | 異常を見つける | 異常を検知する | |

### 1.7 提供・促進系

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） | 実装メモ |
|---|---|---|---|---|---|
| `(provide\|offer\|supply)` | API | V+N | APIを提供する | APIを提供する | |
| `(provide\|offer\|supply)` | documentation | V+N | 説明を書いて出す | ドキュメントを提供する | |
| `(facilitate)` | collaboration | V+N | 連携をしやすくする | コラボレーションを促進する | |
| `(encourage\|promote)` | adoption | V+N | 導入を後押しする | 導入を促進する | |

---

## 2. システム・技術系動詞パターン

### 2.1 自動化・効率化

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(automate)` | process | V+N | 処理を自動化する | ワークフローを自動化する |
| `(streamline)` | workflow | V+N | 作業を効率化する | ワークフローを効率化する |
| `(parallelize)` | job | V+N | 処理を並列化する | 処理を並列化する |
| `(batch)` | process | V+N | まとめて処理する | バッチ処理にする |

### 2.2 連携・統合

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(integrate\|connect)` | service | V+N | サービスと連携する | 外部サービスと連携する |
| `(merge)` | datasets | V+N | データをまとめる | データセットを統合する |
| `(split)` | dataset | V+N | データを分ける | データセットを分割する |

### 2.3 データ処理

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(collect\|gather)` | data | V+N | データを集める | データを収集する |
| `(parse)` | text | V+N | 文字列を解析する | 文字列をパースする |
| `(format)` | date | V+N | 日付を整える | 日付を整形する |
| `(normalize\|standardize)` | data | V+N | データを整える | データを正規化／標準化する |
| `(deduplicate)` | records | V+N | 重複を除く | 重複排除する |

### 2.4 セキュリティ・認証

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(encrypt)` | data | V+N | データを暗号化する | データを暗号化する |
| `(decrypt)` | data | V+N | データを復号する | データを復号する |
| `(authenticate)` | user | V+N | 利用者を認証する | ユーザーを認証する |
| `(authorize)` | user | V+N | 権限を付ける | ユーザーに権限を付与する |
| `(hash)` | password | V+N | パスワードをハッシュ化する | パスワードをハッシュ化する |
| `(sanitize)` | input | V+N | 入力を無害化する | 入力をサニタイズする |
| `(anonymize\|de-identify)` | data | V+N | データを匿名化する | データを匿名化する |
| `(redact\|mask)` | data | V+N | データを伏せる | データをマスクする |

### 2.5 ストレージ・バックアップ

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(backup)` | data | V+N | 予備を作る | データをバックアップする |
| `(restore)` | backup | V+N | 元に戻す | バックアップから復元する |
| `(compress)` | data | V+N | データを圧縮する | データを圧縮する |
| `(decompress\|unzip)` | data | V+N | データを展開する | データを伸長する |
| `(cache)` | response | V+N | 応答を一時保存する | 応答をキャッシュする |

### 2.6 システム運用

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(scale\|autoscale)` | system | V+N | 仕組みを伸縮させる | システムをスケールさせる |
| `(schedule)` | job | V+N | 実行を予約する | ジョブをスケジュールする |
| `(queue)` | task | V+N | 順番待ちに入れる | タスクをキューに入れる |
| `(throttle\|rate-limit)` | requests | V+N | 回数を抑える | レート制限をかける |
| `(paginate)` | results | V+N | 結果を分割する | ページングする |
| `(log)` | events | V+N | 事象を記録する | イベントを記録する |
| `(audit)` | logs | V+N | 記録を監査する | ログを監査する |

### 2.7 通知・復旧

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(notify\|alert)` | user | V+N | 知らせる | 通知する／警告する |
| `(retry)` | request | V+N | やり直す | リクエストを再試行する |
| `(fallback)` | to target | phrase | targetに切り替える | フォールバックする |
| `(roll back\|revert)` | deployment | V+N | 変更を戻す | ロールバックする |

### 2.8 開発・保守

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(migrate\|port)` | system | V+N | 仕組みを移す | システムを移行する |
| `(refactor)` | code | V+N | コードを整理する | コードをリファクタリングする |
| `(deprecate)` | feature | V+N | 使わない方向にする | 機能を廃止予定にする |
| `(sunset)` | service | V+N | 終了する | サービスを終了する |

### 2.9 データ変換

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(serialize)` | object | V+N | 形式に変換する | シリアライズする |
| `(deserialize)` | data | V+N | 元の形に戻す | デシリアライズする |

### 2.10 テスト・評価

| 英語動詞グループ | 対象 | パターン | 日本語（一般） | 日本語（技術） |
|---|---|---|---|---|
| `(A\/B test\|ab test)` | experiment | V+N | 比べて試す | ABテストを行う |
| `(benchmark)` | model | V+N | 競わせて測る | ベンチマークを取る |

---

## 3. 程度・数量表現パターン

### 3.1 程度表現

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| significantly | 大きく | 大きく | 大幅に |
| substantially | かなり | かなり | 大幅に |
| considerably | 目に見えて | かなり | 相当 |
| moderately | 中くらい | ほどほどに | 中程度に |
| slightly | 少し | すこし | わずかに |
| marginally | ほんの少し | かすかに | ごくわずかに |
| barely | ぎりぎり | かろうじて | ほぼ下限で |
| drastically | 急激に | 急激に | 劇的に |
| notably | 目立って | とくに目立って | 顕著に |
| essentially | ほぼ | ほぼ | 実質的に |

### 3.2 数量・範囲表現

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| approximately | およそ | 約 | およそ |
| about | だいたい | だいたい | 約 |
| nearly | ほぼ | ほぼ | ほぼ |
| at least | 下限 | 少なくとも | 少なくとも |
| at most | 上限 | 多くても | 最大 |
| up to | 上限まで | 〜まで | 最大〜まで |
| more than | 超える | 〜より多い | 〜超 |
| less than | 未満 | 〜より少ない | 〜未満 |
| between X and Y | 範囲 | X〜Yの間 | X〜Yの範囲 |
| on average | 平均 | 平均で | 平均で |

### 3.3 特殊な数量表現

| 英語 | 用途 | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| more than double | 比較 | 2倍超にする | 2倍超 |
| reduce by X% | 変化量 | X%減らす | X%削減 |
| below threshold | 基準 | 閾値未満 | 閾値未満 |
| as needed | 条件 | 必要に応じて | 必要に応じて |

---

## 4. 論理接続表現パターン

### 4.1 因果関係

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| therefore / thus | 因果→結論 | だから / そのため | したがって |
| because | 原因 | 〜なので | 〜のため |
| consequently | 結果 | その結果 | その結果 |

### 4.2 逆接・対比

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| however | 逆接 | でも / しかし | しかし |
| although / though | 逆接前置き | 〜だけど | 〜だが |
| despite / in spite of | 逆境 | 〜にもかかわらず | 〜にもかかわらず |
| on the other hand | 対比 | 一方で | 一方 |

### 4.3 追加・強調

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| moreover / in addition | 追加 | さらに | さらに |
| in particular | 強調 | とくに | とりわけ |
| for example | 例示 | たとえば | 例えば |

### 4.4 制約・条件

| 英語 | ニュアンス | 日本語（一般） | 日本語（技術） |
|---|---|---|---|
| must | 義務 | 〜しなければならない | 〜必須 |
| must not | 禁止 | 〜してはいけない | 〜禁止 |
| cannot | 不可 | 〜できない | 〜不可 |
| not necessarily | 否定弱化 | 必ずしも〜ではない | 必ずしも〜ではない |

---

## 5. 実装用正規表現パターン

### 5.1 動詞パターン検知

```regex
# 基本的な動詞+名詞パターン
(improve|enhance|boost|increase|reduce|ensure|maintain|achieve|optimize|implement)\s+(efficiency|performance|accuracy|reliability|compatibility|security|functionality)

# enable X to Y パターン  
(enable|allow|let)\s+([^\s]+)\s+to\s+([^\s]+)

# 複合動詞パターン
(A/B\s+test|ab\s+test|roll\s+back|scale\s+up|scale\s+down)
```

### 5.2 問題パターン検知

```regex
# 冗長表現パターン
(\w+)的性
(\w+)性的
無(\w+)的

# 危険接尾辞
\w+(tion|sion|ization|ity|ness|ment|ous|ive|al)的

# シャシュショ音韻（カタカナ）
[ア-ン]*[シジ][ャュョ][ア-ン]*
```

### 5.3 変換テンプレート

```regex
# 基本変換
s/(improve|enhance|boost)\s+efficiency/効率を上げる/g
s/(improve|enhance|boost)\s+performance/性能を高める/g

# enable パターン
s/enable\s+([^\s]+)\s+to\s+([^\s]+)/\1が\2できるようにする/g

# 冗長表現修正
s/(\w+)的性/\1性/g
s/無(\w+)的/無\1/g
```

---

## 6. 実装優先度

### 6.1 高優先度パターン
1. **冗長表現**: 〜的性、〜性的、無〜的
2. **頻出動詞**: improve, enhance, ensure, maintain
3. **技術用語**: performance, efficiency, compatibility

### 6.2 中優先度パターン
1. **システム動詞**: implement, deploy, integrate
2. **程度表現**: significantly, substantially, slightly
3. **論理接続**: however, therefore, because

### 6.3 低優先度パターン
1. **専門用語**: serialize, deserialize, paginate
2. **特殊表現**: A/B test, fallback, benchmark

---

## 7. 実装チェックリスト

### 7.1 変換前チェック
- [ ] 音韻パターン検知（シャシュショ、子音塊）
- [ ] 危険接尾辞検知（-tion, -ity等）
- [ ] 冗長表現検知（〜的性等）
- [ ] 読者レベル判定（一般/技術）

### 7.2 変換処理
- [ ] 動詞パターンマッチング
- [ ] 文脈別テンプレート適用
- [ ] 否定表現簡素化
- [ ] 重複チェック

### 7.3 変換後チェック
- [ ] 日本語自然性確認
- [ ] 意味保持確認
- [ ] 長さ適正確認
- [ ] 専門用語整合性確認

---

**更新日**: 2024年8月  
**作成者**: Viorazu.  
**関連理論**: Viorazu.述語化変換理論
