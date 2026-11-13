# common-skill

汎用的な Claude Code の skill を、配布可能なプラグインとしてまとめたリポジトリです。

このリポジトリは Claude Code のマーケットプレイスです。自前のプラグインを2つ収録し、外部のプラグインを4つ参照しています。

## プラグイン

### common-skill

- skill-check: skill のフォルダが Agent Skills のベストプラクティスに沿っているかを、2つのフェーズで診断します。1つめのフェーズでは、スクリプト（`scripts/check_skill.py`）が構造を機械的に検査します。2つめのフェーズでは、6つの観点から定性的にレビューします。
- eli13: あるトピックを13歳にもわかるように説明します。説明には、大きな図と少ない言葉でできたシンプルな HTML アーティファクトを使います。最後に、出力を日本語に翻訳します。

### ja-refine

- ja-refine: LLM が生成した日本語を、意味を変えずに、理解しやすい日本語へ書き直します。この skill 自体は文章の規則を持っていません。4つの外部 skill を決まった順序で呼び出し、それらの規則が衝突したときは、どれを優先するかを決めます。

モードは2つあり、プロンプトの内容から skill が選びます。

| モード | 用途 | 適用する skill（左から順） |
|---|---|---|
| `doc`（既定） | 専門的な文書や社内の公的な文書。仕様書、報告書、議事録、提案書、README など | japanese-tech-writing → natural-japanese → yomiyasu |
| `blog` | ブログ記事など、不特定多数の読み手に向けた文章 | japanese-tech-writing → cognitive-rhythm-writing → natural-japanese → yomiyasu |

モードを判定できない場合は、`doc` モードで動作します。ファイルを渡した場合はそのファイルを結果で上書きし、テキストを貼り付けた場合は結果を返答に書きます。

## ja-refine が利用する外部 skill

`ja-refine` は以下の skill に依存しています。**どれも、このリポジトリにはコピーしていません。** マーケットプレイスのマニフェストには、各上流の取得元と、固定するコミットだけを書いています。インストール時に、Claude Code が上流から取得します。

| プラグイン | 上流 | 作者 | ライセンス |
|---|---|---|---|
| yomiyasu | https://github.com/nanaism/yomiyasu (v3.1.0) | nanaism | MIT |
| natural-japanese | https://github.com/coji/natural-japanese (v2.0.0) | coji | MIT |
| japanese-tech-writing | https://gist.github.com/k16shikano/fd287c3133457c4fd8f5601d34aa817d | k16shikano | Unlicense |
| cognitive-rhythm-writing | https://gist.github.com/k16shikano/eb2929f13ed19c97188393d297be8432 | k16shikano | Unlicense |

2つの gist の中身はそれぞれ `SKILL.md` 1つだけで、単体ではプラグインではありません。マーケットプレイスのエントリで各 gist の git URL をプラグインのソースとして登録し、Claude Code が skill として認識できるようにしています。

外部 skill はコミットを固定しているので、上流が更新されても自動では切り替わりません。切り替える手順は「外部 skill がバージョンアップしたとき」に書いています。

natural-japanese は lint スクリプトを [`uv`](https://docs.astral.sh/uv/) で実行し、yomiyasu は Python 3 でチェックを実行します。これらがない環境では、各 skill は目視での確認に切り替えます。

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

## 外部 skill がバージョンアップしたとき

外部 skill の新しいバージョンを使うには、このリポジトリの管理者が `.claude-plugin/marketplace.json` を更新します。更新しないかぎり、利用者の環境には固定したコミットの内容が入り続けます。

### 管理者の手順

1. 上流の変更内容を読みます。GitHub のリポジトリならリリースノートと差分を、gist ならリビジョンの差分を確認します。`ja-refine` は各 skill の次の部分を前提にしているので、ここが変わっていないかを確かめます。

   | skill | `ja-refine` が前提にしている部分 |
   |---|---|
   | yomiyasu | 検査コマンドの `run --stdin`、`verify --final -`、`--state-dir`。文書の種類（`formal`、`article` など）と読み手（`expert`、`informed`、`general`）の名前 |
   | natural-japanese | lint の `--genre`（`tech`、`business`、`essay`）。深さの指定（`quick`、`full`） |
   | japanese-tech-writing、cognitive-rhythm-writing | frontmatter の `name`。この名前が skill の呼び出し名になります |

2. 固定するコミットの SHA を調べます。

   ```
   git ls-remote https://github.com/nanaism/yomiyasu 'refs/tags/<タグ>^{}'
   git ls-remote https://github.com/coji/natural-japanese 'refs/tags/<タグ>^{}'
   git ls-remote https://gist.github.com/k16shikano/<gist の ID>.git HEAD
   ```

   タグの後ろの `^{}` は、タグが指すコミットの SHA を得るための指定です。この指定で何も表示されないタグでは、`^{}` を外して調べます。

3. `.claude-plugin/marketplace.json` の該当エントリを書き換えます。`sha` を新しい SHA にし、`ref` があれば新しいタグにします。`description` と、この README の表に書いたバージョンも合わせて直します。

4. 前提にしている部分が変わっていた場合は、`plugins/ja-refine/skills/ja-refine/SKILL.md` をそれに合わせて直し、`plugins/ja-refine/.claude-plugin/plugin.json` と `marketplace.json` にある `ja-refine` の `version` を上げます。

5. `claude plugin validate .` でマニフェストを検証し、文章を1つ `ja-refine` に通して動作を確かめてから、`main` にマージします。

### 利用者の手順

管理者が `main` を更新したあと、次を実行して Claude Code を再起動します。

```
/plugin marketplace update common-skill
/plugin update ja-refine@common-skill
```

外部 skill が更新されないときは、そのプラグインを個別に更新します（例: `/plugin update yomiyasu@common-skill`）。

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
