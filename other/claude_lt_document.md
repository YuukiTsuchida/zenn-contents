---
marp: true
theme: gaia
class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
style: |
  section {
    font-family: 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', 'Yu Gothic Medium', 'Meiryo', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    text-align: center;
    justify-content: center;
    align-items: center;
  }
  ul, ol {
    text-align: left;
    display: inline-block;
  }
  h1 {
    color: #2563eb;
    border-bottom: 3px solid #3b82f6;
    padding-bottom: 0.5rem;
  }
  h2 {
    color: #1e40af;
    border-left: 4px solid #3b82f6;
    padding-left: 1rem;
  }
  code {
    background-color: #f1f5f9;
    color: #334155;
    padding: 0.2rem 0.4rem;
    border-radius: 0.25rem;
  }
  pre {
    background-color: #1e293b;
    color: #f8fafc;
    border-radius: 0.5rem;
    padding: 1rem;
    border: 1px solid #334155;
    text-align: left;
    font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
    font-size: 0.9em;
    white-space: pre;
    overflow-x: auto;
  }
  pre code {
    background-color: transparent;
    color: #f8fafc;
    padding: 0;
    font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  }
  table {
    border-collapse: collapse;
    margin: 1rem 0;
  }
  th, td {
    border: 1px solid #cbd5e1;
    padding: 0.5rem 1rem;
  }
  th {
    background-color: #e2e8f0;
    font-weight: bold;
  }
---

# Claude CodeでLT資料を作ってみた

---

## 今日のお話

Claude Codeで資料作成実験をやってみました

---

## 実験設定

**人間の役割**: 構成やボリュームのチェックのみ  
**Claude Codeの役割**: 執筆・修正・提案を全て担当

**お題**: JWT (JSON Web Token) の基礎解説資料

---

## まずは成果物をご覧ください

**Claude Codeが作成したJWT解説**をお見せします

「AIがどの程度の資料を作れるか」を体感してください

---

# JWT (JSON Web Token) 基礎
## Claude Code作成版

---

## JWT とは何か？

情報を安全に送信するための公開標準仕様

URLセーフな文字列として表現され、デジタル署名されている

---

### JWTが使われる場面

- **API 認証**: RESTful API の認証
- **シングルサインオン (SSO)**: 複数サービス間の認証
- **マイクロサービス**: サービス間の認証
- **モバイルアプリ**: ネイティブアプリの認証

---

### JWT が解決する問題
```
従来の問題:
「このリクエストは本当に認証済みユーザーからのものか？」
「このユーザーにはどんな権限があるのか？」
「サーバー間でユーザー情報をどう受け渡すか？」

JWT の解決策:
署名付きトークンを使い、「誰が」「どんな権限で」アクセスしているかを
トークン単体で検証可能にし、サーバー間で安全に受け渡し可能
```

---

### 重要な誤解の解消
- **❌ 暗号化ツールではない**: 内容を隠すためのものではない
- **❌ 秘匿性は提供しない**: 誰でも内容を読むことができる
- **✅ 完全性を保証**: 署名により改ざんを検出
- **✅ 真正性を保証**: 正当な発行者からのトークンであることを証明

---

## JWT の構造

### 3つの部分で構成
```
xxxxx.yyyyy.zzzzz
```

1. **Header (ヘッダー)**
2. **Payload (ペイロード)**
3. **Signature (署名)**

---

### 最終的なJWTの形式
```
Base64URL(Header) . Base64URL(Payload) . Base64URL(Signature)
```

---

### Header (ヘッダー)
```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```
- `alg`: 使用する署名アルゴリズム
- `typ`: トークンのタイプ

---

### Payload (ペイロード)
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**注意**: 機密情報（パスワード、クレジットカード番号等）は入れない

---

### Signature (署名)

Base64URLでエンコードしたHeader と Payload をピリオド（.）で結合し、秘密鍵で署名した値

---

## JWT使用時の注意点

- **長期間の認証**: リフレッシュトークンと併用推奨
- **リアルタイム無効化が必要**: セッション管理の方が適している
- **署名検証必須**: `"alg": "none"`攻撃などに注意
- **有効期限チェック**: `exp`クレームを必ず確認

---

## 以上、Claude Code製のJWT解説でした

どう作ったか紹介します

---

## 資料作成のフロー

### 1. 叩き台の作成
「LTでJWTの基礎的な話をするので資料を作ってください」

↓

大雑把な指示で全体の叩き台を出力

---

### 2. 反復的な修正指示
最初のインプットが抽象的だったため、発表の方向性や詳細が不足

↓

「この部分をこう修正して」といった具体的な指示を繰り返し

---

### 実際の修正例

**私**: 「JWTの構造説明が分かりにくいです」  
**Claude**: Header、Payload、Signatureの順で詳細化

**私**: 「セキュリティの懸念について詳しく」  
**Claude**: `"alg": "none"`攻撃などの実装問題と対策を追加

---

## AIに資料作成させて分かった課題

### 全体最適化の限界
部分的な修正は得意だが、**全体を常にチェックしているわけではない**

---

### 具体的な問題例
- **情報の重複**: 異なるセクションで同じ内容を説明
- **構成の不整合**: セクション削除後の番号更新漏れ
- **情報の散在**: 関連する情報が離れた場所に配置
- **局所最適**: 修正依頼部分は完璧だが全体影響を見落とし

---

## 一方で、良かった点も多数

### 作業履歴の記録
- 作成過程の履歴を記憶
- 「なぜこの修正をしたのか」を振り返り可能
- 指示の方向性を見直せる

**ただし**: セッション終了で全て忘れてしまう制限あり

---

### 会話ベースでの資料作成
- 自分の懸念点を伝えるだけで適切な言葉を提案
- リアルタイムで対話しながら改善
- **曖昧な指示も理解**: 「もう少し分かりやすく」等

---

## Claude Codeの特徴まとめ

### 強み
- **コンテンツ執筆**: 文章作成、表現の調整が得意
- **反復修正**: 細かい調整を繰り返す作業に最適
- **一貫性**: 既存の内容との整合性を保つ

---

### 弱み
- **全体俯瞰**: 部分修正時の全体影響を見落としがち
- **セッション依存**: 新しいセッションでは前回の判断基準を忘れる
- **全体設計**: 部分修正の積み重ねでは一貫した構造になりにくい

---

## 今後の改善アイデア

### 構成設計
今回は完全に叩き台から作成したが、**セクション構成程度は事前に人間が作成**してから依頼した方が修正回数を減らせそう

---

### セッション継続の課題解決
「セッション終了で忘れる」制限を解決する方法：

- **CLAUDE.mdの活用**: 重要な判断基準や設計思想を記録
- **Gitコミットログ**: 詳細なコミットメッセージで変更理由を記録

---

## メリット・注意点

**メリット**:
- 実際の執筆作業から解放
- 構成や方向性の検討に集中できる
- 日本語の言い回しや表現を考える負担が激減

---

**注意点**:
- 指示は具体的に
- 全体チェックは人間が担当
- 最終的な品質管理は必須

---

## 感想

体験としては「**有能な部下ができた**」感覚に近いものでした。

指示を出すと期待以上の文章が返ってくる体験は新鮮で、思考を言葉にする負担が大幅に減りました。