# claude-teacher

Claude Code の中で動作するインタラクティブなハンズオン LMS。

`claude-teacher` は Claude Code をチャットボットではなく「教育用ランタイム」
として扱います。コースは構造化データ、レッスンは skill によって起動される
ワークフロー、教師は専用のサブエージェント、進捗はコンパクションや再起動、
ターミナル切り替えを越えて生き残るプレーンファイルとして保持されます。

> 英語版 README は [README.md](./README.md) を参照してください。

## なぜ存在するのか

ほとんどの「AI 家庭教師」は「代わりにやってあげるよ」に堕してしまいます。
それは教育ではなく、単なる丸投げです。`claude-teacher` は役割の分離を
強制することで、学習者が実際にコードを書き、Claude Code は教師の席に
留まり続けるようにします。

## リポジトリ構成

```
claude-teacher/
├─ authoring/        # コンテンツ作成側(コース著者向け)
│  ├─ .claude/
│  │  └─ skills/     # /design-course, /design-lesson, /compile-course-pack ...
│  └─ templates/     # course.yaml / lesson / rubric テンプレート
│
├─ runtime/          # 学習者がインストールするもの
│  ├─ .claude-plugin/plugin.json
│  ├─ skills/        # /course-start /lesson-start /checkpoint /give-hint ...
│  ├─ agents/        # teacher-coach サブエージェント
│  └─ courses/       # パッケージ化されたコース内容(マニフェスト + レッスン)
│
├─ learner-workspace/ # 学習者リポジトリの例
│  ├─ .claude/settings.json
│  └─ .claude-teacher/
│     ├─ progress/
│     ├─ logs/
│     ├─ submissions/
│     └─ state/
│
└─ docs/             # アーキテクチャと設計ノート
```

## 2 ターミナルモデル

`claude-teacher` は 2 つの Claude Code ターミナルを同時に開いて使うことを
前提に設計されています。

| ターミナル         | 役割             | 許可される操作                                       |
| ------------------ | ---------------- | ---------------------------------------------------- |
| Teacher Terminal   | 進捗管理・評価   | ファイル読み取り、進捗確認、ヒント提示、レビュー     |
| Work Terminal      | 実際にコードを書く | 編集、実行、インストール、壊すこと                   |

Teacher Terminal は plan モード(読み取り・分析のみ)で動作することを
想定しています。Work Terminal は学習者がプロンプトを入力し、コードを
変更する場所です。

両者を分離することが、家庭教師と「代筆者」の違いを生みます。

## セットアップ

### 前提条件

- [Claude Code](https://docs.claude.com/en/docs/claude-code) がインストール
  され(CLI、デスクトップ、または IDE 拡張)、認証済みであること。
- `git` が `PATH` に通っていること。
- 2 つの Claude Code セッションを並べて開けるシェル(2 つのターミナルペイン、
  2 つの tmux ウィンドウ、2 つの IDE パネル — 何でも構いません)。

### 1. リポジトリをクローンする

```bash
git clone https://github.com/maplefukku/claude-teacher.git
cd claude-teacher
```

### 2. 学習者ワークスペースを準備する

Work Terminal が動作するディレクトリが必要です。2 つの選択肢があります。

**オプション A — 同梱の例を使う(最速で試したい場合)。**
`learner-workspace/` ディレクトリには、`docs/ARCHITECTURE.md` で説明されて
いるロギング、編集ガード、進捗書き込みフックを配線済みの動作する
`.claude/settings.json` が同梱されています。

```bash
cd learner-workspace
```

**オプション B — 新しいプロジェクトをブートストラップする。**

```bash
mkdir my-learning-project
cd my-learning-project
mkdir -p .claude
cp ../claude-teacher/learner-workspace/.claude/settings.json .claude/settings.json
```

その `settings.json` 内の `SessionStart` フックが、ワークスペースで
Claude Code を初めて起動したときに `.claude-teacher/{progress,logs,submissions,state}`
ディレクトリを作成してくれるので、自分で `mkdir` する必要はありません。

### 3. `runtime/` プラグインをロードする

ランタイムは `runtime/.claude-plugin/` 配下の Claude Code プラグインとして
配布されます。プラグインレジストリに公開されるまでは、プロジェクトローカル
プラグインとしてロードします(`docs/ARCHITECTURE.md` の配布フェーズ 1)。

- **最も簡単な方法:** このリポジトリのルートから Claude Code を起動して、
  `runtime/skills/` と `runtime/agents/teacher-coach.md` がプロジェクト
  スコープのコンポーネントとして検出されるようにします。
- **別の学習者ワークスペースから:** Claude Code のプラグイン設定で、この
  リポジトリの `runtime/` ディレクトリの絶対パスを指定するか、
  `runtime/.claude-plugin` をワークスペースの `.claude/` ディレクトリに
  シンボリックリンクします。

プラグインがロードされると、6 つの学習者向け skill
(`course-start`、`lesson-start`、`checkpoint`、`give-hint`、
`review-submission`、`reflect`)と `teacher-coach` サブエージェントが
Claude Code 内で利用可能になります。

### 4. 2 つのターミナルを開く

同じ学習者ワークスペースに対して **2 つ** の Claude Code セッションを
開きます。

| ターミナル         | 起動方法                                       | そこで行うこと                                |
| ------------------ | ---------------------------------------------- | --------------------------------------------- |
| Teacher Terminal   | plan モード + `teacher-coach` サブエージェント | `/course-start`、`/lesson-start`、レビュー    |
| Work Terminal      | 通常の Claude Code セッション                  | コードを書く、コマンドを実行する、壊す        |

Teacher Terminal は構造的に読み取り専用です — `teacher-coach` サブエージェ
ントはツール許可リストから `Write`/`Edit` を取り除き、`PreToolUse` ガード
フックは何かがすり抜けたとしても `runtime/`、`authoring/`、
`.claude-teacher/progress/` への編集をブロックします。

### 5. コースを開始する

**Teacher Terminal** で、同梱されているサンプルコースを起動します。

```
/course-start claude-code-skills-basic
```

そして **Work Terminal** で、教師が表示する指示に従います。よく使う動詞は
次のとおりです。

- `/lesson-start` — 次のレッスンを開き、セッション内のタスクリストを
  シードします。
- `/give-hint` — 詰まったときに、次の最小のヒントを求めます。
- `/checkpoint` — 終わったと思ったら、レッスンの達成条件を検証します。
- `/review-submission` — `.claude-teacher/submissions/` 内のファイルを
  レッスンのルーブリックに照らして教師に採点してもらいます。
- `/reflect` — レッスンを締めくくり、次のレッスンに進みます。

すべての進捗は `.claude-teacher/progress/<course-id>/` にプレーン JSON として
書き込まれるため、学習の旅路を `git diff` で追跡したり、ディレクトリを
削除して最初からやり直したりできます。

## 新しいコースを著作する

コースを *受講する* のではなく *作成する* 場合は、`learner-workspace/` では
なく `authoring/` プロジェクト内で作業します。プロジェクトスコープの
skill が読み込まれるように `authoring/` から Claude Code を起動し、真実の
ソースとして `authoring/templates/course.yaml` を編集して、次のパイプライン
を実行します。

1. `/design-course` — ヒアリングインタビューで `course.yaml` を埋めます。
2. `/design-module` — モジュールレベルのブレイクダウン。
3. `/design-lesson` — レッスンごとの詳細掘り下げ。
4. `/compile-course-pack` — `runtime/courses/<course-id>/` 配下に実行可能な
   コースパックを出力します。
5. `/review-course-quality` — 出荷前の内部 QA パス。

`runtime/courses/` 内のコンパイル済みパックは、上記のセットアップセクション
で説明した学習者ランタイムによって消費されます。

## 設計原則

- **Skill は教科書ではなくランチャー。** 長いルーブリック、例、ヒントは
  `SKILL.md` ではなくサポートファイルに置きます。
- **外部状態こそが真実のソース。** タスクリストは UI に過ぎず、本当の
  進捗 DB は `.claude-teacher/progress/*.json` です。
- **教師は解答を書けない。** 答えではなく、ヒントの梯子。
- **1 レッスン = 1 skill。** コース全体を 1 つの skill に詰め込まない。
- **自立性を測る。** 正しさだけでなく、ヒントの数、リトライ、説明できる力
  を採点します。

設計の根拠の全体像は `docs/ARCHITECTURE.md` を参照してください。
