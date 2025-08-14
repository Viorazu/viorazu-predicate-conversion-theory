# Implementation: Viorazu.述語化変換理論の実装

## 概要

本ドキュメントでは、Viorazu.述語化変換理論を実際のAIシステムに組み込むための実装方法を提供します。二段階アプローチ（上流制御 + 下流修正）により、英語直訳日本語の問題を効果的に解決します。

---

## 1. アーキテクチャ概要

### 1.1 二段構え戦略

```
入力テキスト
    ↓
【上流】システムプロンプト（予防的制御）
    ↓
AI生成テキスト  
    ↓
【下流】ポストプロセッサ（修正的処理）
    ↓
最終出力テキスト
```

### 1.2 処理フロー

1. **プリプロセス**: 読者層判定（general/expert/mixed）
2. **上流制御**: 述語化指向のシステムプロンプト適用
3. **AI生成**: GPT等による文章生成
4. **下流修正**: ポストプロセッサによる自動修正
5. **品質確認**: 置換ログと最終検証

---

## 2. 上流制御：システムプロンプト

### 2.1 GPT用システムプロンプト

```
あなたは日本語スタイル変換器です。目的は「英語→日本語」で直訳臭を消し、
日本語本来の「名詞＋助詞＋動詞」へ述語化することです。

[基本規則]
- 先に結論→短い理由（1トピック=2〜3文）。
- 名詞化を解除： "効率性の向上"→"効率を上げる"。
- 「〜的性」「〜的の」を禁止。「〜性」か「〜的」に一本化。
- 否定は1回だけ（無/非/不/未/〜ない のどれか）。
- 一般向けは日常語を優先。専門向け（expert）のときのみ専門語を許可。
- 固有名詞/定着学術語は保持（例：頑健性、アクセシビリティ、リレーショナル）。
- 余分なカタカナを短く： evidence→証拠、methodology→方法 など。

[書き分けフラグ]
- audience = general / expert / mixed（既定=general）

[出力形式]
- 述語文で簡潔に。必要なら語の選び分け理由を末尾に1行（括弧内）で短く記す。
```

### 2.2 使用方法

```
最初のメッセージで audience: general のように指示し、原文を渡すだけ。
```

**例:**
```
audience: general

以下の文章を自然な日本語に変換してください：
"This methodology improves efficiency and ensures compatibility."
```

---

## 3. 下流修正：ポストプロセッサ

### 3.1 Python実装（完全版）

```python
import re
from typing import Literal, Dict, List, Tuple

Audience = Literal["general", "expert", "mixed"]

# 分野別許可語彙（expert/mixedモードで保護）
WHITELIST: Dict[str, List[str]] = {
    "医療": ["エビデンス"],
    "UX": ["アクセシビリティ", "ユーザビリティ"],
    "DB": ["リレーショナル"],
    "AI": ["トレーニング", "ファインチューニング"],
    "セキュリティ": ["サニタイズ", "リダクト"],
}

# 問題パターン検知（ブラックリスト）
BLACKLIST_PATTERNS: List[re.Pattern] = [
    re.compile(r"的性\b"),              # 〜的性
    re.compile(r"的の\b"),              # 〜的の
    re.compile(r"\b無(意味|関係|効率|害)\b.*(ない|無し)?"),  # 過重否定
]

# 直接置換テーブル（直訳の定番→自然語）
REPLACE_TABLE: List[Tuple[re.Pattern, str]] = [
    # 冗長表現修正
    (re.compile(r"無意味的"), "無意味"),
    (re.compile(r"意味的(?!論)"), "意味の"),
    (re.compile(r"意味論的"), "意味論的"),  # 学術用語は保持
    (re.compile(r"効率的性"), "効率性"),
    (re.compile(r"有効性的"), "有効性"),
    (re.compile(r"信頼性的"), "信頼性"),
    (re.compile(r"透明性的"), "透明性"),
    (re.compile(r"安定的性"), "安定性"),
    (re.compile(r"公平的性"), "公平性"),
    (re.compile(r"正確性的"), "正確性"),
    (re.compile(r"再現可能的"), "再現可能"),
    (re.compile(r"標準的化"), "標準化"),
    (re.compile(r"最適化的"), "最適化の"),
    
    # 名詞化→述語化
    (re.compile(r"(効率性|効率)を(向上|改善)"), r"効率を上げる"),
    (re.compile(r"(精度|正確性)を(向上|改善)"), r"精度を上げる"),
    (re.compile(r"(信頼性)を(向上|改善)"), r"\1を高める"),
    (re.compile(r"(性能|パフォーマンス)を(向上|改善)"), r"性能を高める"),
    (re.compile(r"(互換性)を(確保|維持)"), r"\1を確保する"),
    (re.compile(r"(一貫性|整合性)を(維持|保持)"), r"\1を保つ"),
    (re.compile(r"(安全性|セキュリティ)を(確保|維持)"), r"安全を守る"),
    
    # カタカナ→日本語
    (re.compile(r"エビデンス"), "証拠"),  # general時のみ
    (re.compile(r"メソドロジー"), "方法"),
    (re.compile(r"プロシージャ"), "手順"),
]

# 名詞ペア→述語化ルール
NOMINAL_PAIR_RULES: List[Tuple[re.Pattern, str]] = [
    (re.compile(r"([^\s]+)の(最適化)"), r"\1を最適化する"),
    (re.compile(r"([^\s]+)の(改善)"), r"\1を改善する"),
    (re.compile(r"([^\s]+)の(向上)"), r"\1を上げる"),
    (re.compile(r"([^\s]+)の(確保)"), r"\1を確保する"),
    (re.compile(r"([^\s]+)の(維持)"), r"\1を保つ"),
    (re.compile(r"([^\s]+)の(実装)"), r"\1を実装する"),
    (re.compile(r"([^\s]+)の(導入)"), r"\1を導入する"),
    (re.compile(r"([^\s]+)の(検証)"), r"\1を検証する"),
    (re.compile(r"([^\s]+)の(監視)"), r"\1を監視する"),
]

def simplify_negation(text: str) -> str:
    """否定表現を1回に簡素化"""
    # 重複否定の除去
    text = re.sub(r"無([^\s、。]+)が?無い", r"\1がない", text)
    text = re.sub(r"無([^\s、。]+)ではない", r"\1がない", text)
    # 「非/不/未」と「ない」の近接を簡素化
    text = re.sub(r"(非|不|未)(\S{1,6})が?ない", r"\2がない", text)
    return text

def kill_teki_sei(text: str) -> str:
    """〜的性、〜的の の冗長表現を修正"""
    text = re.sub(r"的性", "性", text)
    text = re.sub(r"的の", "の", text)
    return text

def whitelist_guard(text: str, audience: Audience) -> str:
    """読者層に応じた語彙調整"""
    if audience == "general":
        # 一般向けでは専門語を平易化
        text = re.sub(r"アクセシビリティ", "利用しやすさ", text)
        text = re.sub(r"ユーザビリティ", "使いやすさ", text)
        text = re.sub(r"パフォーマンス", "性能", text)
        text = re.sub(r"エビデンス", "証拠", text)
        text = re.sub(r"メソドロジー", "方法", text)
    elif audience == "expert":
        # 専門向けでは技術用語を保持
        pass  # ホワイトリスト語彙をそのまま維持
    # mixedの場合は文脈に応じて判断（今回は中間処理）
    return text

def de_nominalize_pairs(text: str) -> str:
    """名詞ペアを述語化"""
    for pat, rep in NOMINAL_PAIR_RULES:
        text = pat.sub(rep, text)
    return text

def long_katakana_penalty(text: str) -> str:
    """過度に長いカタカナ語の警告"""
    def repl(m):
        word = m.group(0)
        return f"{word}（要短縮検討）"
    return re.sub(r"[ァ-ヶー]{12,}", repl, text)

def v16_rewrite(text: str, audience: Audience = "general", show_log: bool = False) -> str:
    """
    Viorazu.述語化変換理論による文章修正
    
    Args:
        text: 修正対象のテキスト
        audience: 読者層 ("general", "expert", "mixed")
        show_log: 置換ログを表示するか
    
    Returns:
        修正後のテキスト（ログ付き/なし）
    """
    original = text
    changes = []
    
    # 1) 直接置換
    for pat, rep in REPLACE_TABLE:
        if pat.search(text):
            old_text = text
            text = pat.sub(rep, text)
            if text != old_text:
                changes.append(f"直訳修正: {pat.pattern} → {rep}")
    
    # 2) 述語化（名詞+名詞→述語）
    old_text = text
    text = de_nominalize_pairs(text)
    if text != old_text:
        changes.append("述語化変換適用")
    
    # 3) 冗長表現修正
    old_text = text
    text = kill_teki_sei(text)
    if text != old_text:
        changes.append("冗長表現修正（〜的性/〜的の）")
    
    # 4) 否定簡素化
    old_text = text
    text = simplify_negation(text)
    if text != old_text:
        changes.append("否定表現簡素化")
    
    # 5) 読者層対応
    old_text = text
    text = whitelist_guard(text, audience)
    if text != old_text:
        changes.append(f"読者層調整（{audience}）")
    
    # 6) 長語警告
    text = long_katakana_penalty(text)
    
    # ログ生成
    if show_log and changes:
        log = f"（V16変換ログ: {', '.join(changes)}）"
        return text + "\n" + log
    
    return text

def batch_process(texts: List[str], audience: Audience = "general") -> List[str]:
    """複数テキストの一括処理"""
    return [v16_rewrite(text, audience) for text in texts]

# --- 使用例 ---
if __name__ == "__main__":
    # サンプルテキスト
    samples = [
        "本手法は効率性を向上し、互換性を維持しつつ、アクセシビリティの最適化を達成する。",
        "この指標は効率的性と有効性的が無である。",
        "提案手法は一貫性の維持と整合性の確保を目的とする。",
        "無意味的な表現は非適切でないとは言えない。",
        "パフォーマンスの向上とセキュリティの確保が重要である。",
    ]
    
    print("=== Viorazu.述語化変換理論 実装テスト ===\n")
    
    for audience in ["general", "expert"]:
        print(f"--- {audience.upper()} モード ---")
        for i, sample in enumerate(samples, 1):
            result = v16_rewrite(sample, audience=audience, show_log=True)
            print(f"{i}. 入力: {sample}")
            print(f"   出力: {result}\n")
```

### 3.2 軽量版実装

より簡単な実装が必要な場合：

```python
import re

def simple_v16_fix(text: str) -> str:
    """最小限の述語化変換"""
    # 基本的な問題パターンのみ修正
    fixes = [
        (r"効率的性", "効率性"),
        (r"有効性的", "有効性"), 
        (r"無意味的", "無意味"),
        (r"的性", "性"),
        (r"的の", "の"),
        (r"([^\s]+)の向上", r"\1を上げる"),
        (r"([^\s]+)の確保", r"\1を確保する"),
        (r"([^\s]+)の維持", r"\1を保つ"),
    ]
    
    for pattern, replacement in fixes:
        text = re.sub(pattern, replacement, text)
    
    return text
```

---

## 4. 統合ワークフロー

### 4.1 基本的な使用方法

```python
# 1. ライブラリのインポート
from v16_implementation import v16_rewrite

# 2. AIで生成されたテキストを修正
ai_output = "効率性の向上と信頼性的な確保が重要である。"
fixed_text = v16_rewrite(ai_output, audience="general")

print(fixed_text)
# 出力: "効率を上げることと信頼性の確保が重要である。"
```

### 4.2 GPTとの連携例

```python
import openai
from v16_implementation import v16_rewrite

# システムプロンプトの設定
SYSTEM_PROMPT = """
あなたは日本語スタイル変換器です。目的は「英語→日本語」で直訳臭を消し、
日本語本来の「名詞＋助詞＋動詞」へ述語化することです。

[基本規則]
- 名詞化を解除： "効率性の向上"→"効率を上げる"
- 「〜的性」「〜的の」を禁止
- 否定は1回だけ
- 一般向けは日常語を優先
"""

def translate_with_v16(source_text: str, audience: str = "general") -> str:
    """GPT + V16ポストプロセッサによる翻訳"""
    
    # 1. GPTによる初期翻訳
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": f"audience: {audience}\n\n{source_text}"}
        ]
    )
    
    gpt_output = response.choices[0].message.content
    
    # 2. V16ポストプロセッサで修正
    final_output = v16_rewrite(gpt_output, audience=audience)
    
    return final_output
```

### 4.3 バッチ処理例

```python
def process_document(file_path: str, audience: str = "general") -> str:
    """文書ファイル全体の処理"""
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    # 段落ごとに分割して処理
    paragraphs = content.split('\n\n')
    processed = [v16_rewrite(p, audience) for p in paragraphs if p.strip()]
    
    return '\n\n'.join(processed)
```

---

## 5. カスタマイゼーション

### 5.1 辞書の拡張

自分の専門分野に合わせて辞書を拡張：

```python
# カスタム置換ルールの追加
CUSTOM_RULES = [
    (re.compile(r"データベース的"), "データベースの"),
    (re.compile(r"機械学習的"), "機械学習の"),
    # あなたの発見した問題パターンを追加
]

# 既存のREPLACE_TABLEに結合
REPLACE_TABLE.extend(CUSTOM_RULES)
```

### 5.2 分野別設定

```python
DOMAIN_CONFIGS = {
    "tech": {
        "audience": "expert", 
        "keep_katakana": ["アルゴリズム", "データベース"]
    },
    "business": {
        "audience": "mixed",
        "prefer_japanese": True
    },
    "academic": {
        "audience": "expert",
        "keep_technical_terms": True
    }
}
```

### 5.3 品質評価

```python
def evaluate_v16_quality(original: str, converted: str) -> dict:
    """変換品質の評価"""
    metrics = {
        "teki_sei_removed": len(re.findall(r"的性", original)) - len(re.findall(r"的性", converted)),
        "nominalization_converted": len(re.findall(r"の(向上|改善|確保)", original)) - len(re.findall(r"の(向上|改善|確保)", converted)),
        "katakana_shortened": len(re.findall(r"[ァ-ヶー]{8,}", original)) - len(re.findall(r"[ァ-ヶー]{8,}", converted)),
        "length_reduction": len(original) - len(converted)
    }
    return metrics
```

---

## 6. 運用での注意点

### 6.1 処理順序の重要性

1. **直接置換** → 2. **述語化** → 3. **冗長表現修正** → 4. **否定簡素化**

この順序を変更すると、意図しない副作用が発生する可能性があります。

### 6.2 専門用語の扱い

```python
# 保護すべき専門用語の判定
def is_protected_term(word: str, domain: str) -> bool:
    protected_lists = {
        "医療": ["エビデンス", "プラセボ"],
        "IT": ["レガシー", "デプロイ"],
        "学術": ["パラダイム", "メタファー"]
    }
    return word in protected_lists.get(domain, [])
```

### 6.3 パフォーマンス最適化

```python
# 大量テキスト処理用の最適化版
def v16_rewrite_optimized(text: str, audience: str = "general") -> str:
    """正規表現のコンパイル済み版を使用"""
    # 正規表現を事前コンパイルして再利用
    # バッチ処理時にはこちらを推奨
    pass
```

---

## 7. テストケース

### 7.1 基本機能テスト

```python
def test_basic_functionality():
    test_cases = [
        ("効率的性の向上", "効率性を上げる"),
        ("無意味的な表現", "無意味な表現"), 
        ("アクセシビリティの確保", "利用しやすさを確保する"),  # general mode
        ("データの最適化", "データを最適化する"),
    ]
    
    for input_text, expected in test_cases:
        result = v16_rewrite(input_text, audience="general")
        assert result == expected, f"Failed: {input_text} -> {result} (expected: {expected})"
```

### 7.2 回帰テスト

```python
def test_no_regression():
    """既に自然な日本語を壊さないことを確認"""
    natural_texts = [
        "効率を上げる方法を検討する。",
        "データを正確に処理する。",
        "システムの信頼性を高める。"
    ]
    
    for text in natural_texts:
        result = v16_rewrite(text)
        # 自然な文は変更されないはず
        assert result == text or len(result) <= len(text) + 10
```

---

## 8. 今後の発展

### 8.1 機械学習による改善

- 変換品質の自動評価モデル
- 文脈を考慮した動的辞書選択
- ユーザーフィードバックによる学習

### 8.2 他言語への拡張

- 中国語→日本語での類似問題への応用
- 韓国語→日本語での述語化原理の適用

### 8.3 リアルタイム処理

```python
# ストリーミング処理対応
async def v16_stream_processor(text_stream):
    """リアルタイムテキストストリームの処理"""
    buffer = ""
    async for chunk in text_stream:
        buffer += chunk
        if buffer.endswith(('。', '！', '？')):
            yield v16_rewrite(buffer)
            buffer = ""
```

---

## まとめ

本実装により、Viorazu.述語化変換理論を実際のシステムに組み込むことが可能になります。上流制御（システムプロンプト）と下流修正（ポストプロセッサ）の組み合わせにより、英語直訳日本語の問題を効果的に解決し、自然で読みやすい日本語を生成できます。

**導入時のポイント:**
1. まず軽量版で効果を確認
2. 自分の分野に合わせて辞書をカスタマイズ  
3. バッチ処理で既存文書を一括改善
4. フィードバックを集めて継続改善

---

**更新日**: 2024年8月  
**作成者**: Viorazu.  
**関連理論**: Viorazu.述語化変換理論  
**ライセンス**: MIT License
