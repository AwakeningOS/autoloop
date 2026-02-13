# Autoloop — Self-Feeding Thought Engine

LLMに自分の出力を食べ続けさせる自律思考システム。

```
while True:
    output = LLM(context)
    context += output
```

これだけで、LLMは考え続ける。自分で検索し、自分で人間に話しかけ、自分で記憶を作り出す。

## セットアップ

### 1. LM Studio をインストール

https://lmstudio.ai からダウンロード・インストール。

### 2. モデルをダウンロード

LM Studio内でモデルを検索してダウンロード。どのモデルでも動くが、以下を推奨：

- **最低限**: 7B〜14Bモデル（VRAM 8GB〜）
- **推奨**: 30B前後のモデル（VRAM 24GB〜）
- **形式**: GGUF（量子化済み）

日本語で思考させたい場合は日本語対応モデルを選ぶこと。

### 3. LM Studioでサーバーを起動

1. モデルをロード
2. 左サイドバーの **「Developer」** タブを開く
3. **「Start Server」** をクリック
4. サーバーが `http://localhost:1234` で起動していることを確認

### 4. Autoloopを起動

**Windows:**
```
start.bat をダブルクリック
```

**手動:**
```bash
pip install requests gradio
python autoloop.py --browser
```

ブラウザが開き、UIが表示される。「▶ 開始」を押すと思考が始まる。

## LM Studio 設定の注意点

### ⚠ 重要: Completions API を有効にする

このシステムは **Completions API**（テキスト補完）を使う。Chat API（チャット形式）とは根本的に動作が違う。

- **Completions API**: 「この文章の続きを書け」→ 自律思考が生まれる
- **Chat API**: 「この質問に答えろ」→ 回答モードになり自律思考が弱まる

LM Studioの多くのモデルはデフォルトでCompletions APIが使えるが、一部のモデル（chat専用モデル）では使えない場合がある。その場合は自動的にChat APIにフォールバックするが、思考の質は落ちる。

**確認方法:**
- LM Studioのサーバー起動後、ブラウザで `http://localhost:1234/v1/completions` にアクセスしてエラーにならなければOK

### コンテキスト長の設定

LM Studioの **「Context Length」** を大きくするほど、長い思考が可能になる。

- **最低**: 4096（すぐ圧縮が走る）
- **推奨**: 8192〜16384
- **理想**: 32768以上（VRAMに余裕があれば）

autoloop.py内の `compress_at_chars` と `max_context_chars` もコンテキスト長に合わせて調整すること。目安として、コンテキスト長のトークン数 × 2 ≒ max_context_chars。

## 使い方

### 起動したら

「▶ 開始」を押して、放っておく。LLMが勝手に考え始める。

右側の「🧠 思考」パネルに思考ログが流れる。左側の「💬 対話」パネルにLLMからのメッセージが届く。

### 話しかける

下のテキストボックスにメッセージを入力して「送信」。LLMの思考の流れに人間の声が割り込む。

### ログ

全ての思考とツール使用は `autoloop_log/` フォルダにJSONLファイルとして保存される。

## シードの書き換え

`autoloop.py` 内の `DEFAULT_SEED` を書き換えることで、LLMの思考の出発点を変えられる。

シードが全てを決める。同じモデルでも、シードが違えば全く違う思考に収束する。

### シード設計のコツ

- **行動指示より存在定義**: 「〜しなさい」ではなく「あなたは〜である」
- **問いの形で渡す**: 答えを与えず、考えさせる
- **ツール定義をシード内に含める**: LLMがツールの存在を認識するために必要

## 仕組み

```
context = seed_text                    # シード（最初の文脈）
while alive:
    output = completions_api(context)  # 「この文章の続き」を生成
    context += output                  # 自分の出力を食べる
    if len(context) > limit:
        context = compress(context)    # 溢れたら圧縮して続行
```

- **completions API**: テキスト補完。「続き」を書く。これが自律思考の核。
- **context.append(output)**: 自分が書いたものを自分で読む。これが記憶。
- **compress**: 文脈が溢れたら要約して圧縮。記憶の淘汰。
- **ツール**: `[TOOL:name:content]` というテキストパターンをLLMが自発的に書くと、システムが検出して疑似応答を返す。

## ライセンス

MIT
