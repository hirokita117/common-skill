# common-skill

汎用的な Claude Code の skill を、配布可能なプラグインとしてまとめたリポジトリです。

このリポジトリは Claude Code の**マーケットプレイス**として機能します。自前のプラグインを2つ提供し、さらに外部のプラグインを4つ参照しています。

## プラグイン

### common-skill

- **skill-check** — skill のフォルダが Agent Skills のベストプラクティスに沿っているかを、2つのフェーズで診断します。1つめはスクリプト（`scripts/check_skill.py`）による決定的な構造チェックで、2つめは6つの観点からの定性的なレビューです。
- **eli13** — あるトピックを13歳にもわかるように説明します。大きな図と少ない言葉で構成されたシンプルな HTML アーティファクトを使い、最終的な出力は日本語に翻訳されます。

### ja-refine

- **ja-refine** — LLM が生成した日本語を、意味を変えることなく、理解しやすい日本語へと書き直します。この skill 自体は文章の規則を持っていません。4つの外部 skill を決まった順序でオーケストレーションし、それらの規則が衝突した場合の解決を担います。

モードは2つあり、プロンプトの内容から選択されます。

| モード | 用途 | 適用する段階 |
|---|---|---|
| `doc`（既定） | 専門的な文書や社内の公的な文書。仕様書、報告書、議事録、提案書、README など | japanese-tech-writing → natural-japanese → yomiyasu |
| `blog` | ブログ記事など、不特定多数の読み手に向けて書かれた文章 | japanese-tech-writing → cognitive-rhythm-writing → natural-japanese → yomiyasu |

モードを判定できない場合は、`doc` モードで動作します。ファイルを渡した場合はそのファイルを結果で上書きし、テキストを貼り付けた場合は結果を返答として返します。

## ja-refine が利用する外部 skill

`ja-refine` は以下の skill に依存しています。**これらはいずれも、このリポジトリにコピーされていません。** マーケットプレイスのマニフェストは、コミットを固定したうえで各上流のソースを指し示しているだけであり、Claude Code がインストール時にそこから取得します。

| プラグイン | 上流 | 作者 | ライセンス |
|---|---|---|---|
| yomiyasu | https://github.com/nanaism/yomiyasu (v3.1.0) | nanaism | MIT |
| natural-japanese | https://github.com/coji/natural-japanese (v2.0.0) | coji | MIT |
| japanese-tech-writing | https://gist.github.com/k16shikano/fd287c3133457c4fd8f5601d34aa817d | k16shikano | Unlicense |
| cognitive-rhythm-writing | https://gist.github.com/k16shikano/eb2929f13ed19c97188393d297be8432 | k16shikano | Unlicense |

2つの gist はそれぞれ `SKILL.md` を1つだけ持っており、単体ではプラグインではありません。マーケットプレイスのエントリが各 gist の git URL をプラグインのソースとして登録することで、Claude Code がそれを skill として認識できるようにしています。

上流の新しいバージョンに移行するには、`.claude-plugin/marketplace.json` の該当エントリの `sha`（および `ref`）を更新します。

natural-japanese は lint スクリプトを [`uv`](https://docs.astral.sh/uv/) で実行し、yomiyasu は Python 3 でチェックを実行します。これらがない環境では、各 skill は目視での確認にフォールバックします。

## インストール

```
/plugin marketplace add hirokita117/common-skill
/plugin install common-skill@common-skill
/plugin install ja-refine@common-skill
```

`ja-refine` をインストールすると、上記の4つの外部プラグインも依存関係としてインストールされます。

インストール後は、次のように skill を呼び出します。

```
/common-skill:skill-check [skill フォルダのパス]
/common-skill:eli13 [トピック]
/ja-refine:ja-refine [doc|blog] [ファイルまたは依頼内容]
```

## 構成

```
common-skill/
├── .claude-plugin/
│   ├── plugin.json        # common-skill プラグインのマニフェスト
│   └── marketplace.json   # マーケットプレイスの定義（自前2つ + 外部4つ）
├── skills/                # common-skill プラグインの skill
│   ├── skill-check/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── check_skill.py
│   │       └── list_skills.py
│   └── eli13/
│       └── SKILL.md
└── plugins/
    └── ja-refine/         # ja-refine プラグイン
        ├── .claude-plugin/
        │   └── plugin.json   # 4つの依存関係を宣言
        └── skills/
            └── ja-refine/
                └── SKILL.md
```

`check_skill.py` スクリプトの実行には PyYAML が必要です（`pip install pyyaml`）。
