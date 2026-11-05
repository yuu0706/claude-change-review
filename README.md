# claude-change-review

| 項目 | 内容 |
| --- | --- |
| 対象 | Claude Codeの決まり（CLAUDE.md・共通方針・スキル・メモリ）の直したいことの記録と、会話での反映 |
| 動作環境 | macOS、Xcodeのコマンドラインツール（`/usr/bin/python3`3.9以上の実体）、Claude Code。過去の台帳から戻すときはgit |
| 前提のリポジトリ | 無し。会話の開始時に件数を知らせるには[claude-report-inbox](https://github.com/yuu0706/claude-report-inbox)を併せて入れる |
| ライセンス | MIT（[LICENSE](LICENSE)） |

Claude Codeの決まりの直したいことを、1本のMarkdownの記録に1件ずつ足すスキル`change-review`を配布する。定期実行は記録に足すだけで、対象のファイルを直さない。会話のClaudeが記録を読み、区分に従って反映する。

## 1. このスキルでできること

| できること | 頼み方の例 |
| --- | --- |
| 記録の未対応の項目を、ID・1行の要約・区分・対象で一覧にする | 「直したいことを見せて」 |
| `直してよい`の項目を、差分が今のファイルにそのまま当たるか確かめてから入れる | 「直したいことを見せて」 |
| `要確認`の項目を番号付きで示し、承認されたものだけを入れる | 「反映して」「1と3だけ承認」 |
| 却下した項目を見送りにし、否認リストに理由を残す | 「2は却下。理由は〜」 |
| 入れた変更を直す前の版で戻し、正しい結論を訂正リストに残す | 「<ID>を戻して」 |
| 会話で見つけた決まりの案を、`要確認`として記録に足す | 「この決まりを直したいことに足して」 |

## 2. 使いどころの例

| 頼み方 | Claudeが行うこと |
| --- | --- |
| 「直したいことを見せて」 | `直してよい`は`patch --dry-run -F 0`で当たりを確かめて入れる。`要確認`は番号・要約・理由を並べて示す |
| 「反映して」 | その時点で示した`要確認`のすべてを承認とみなす。直す前の版を残してから差分の全体を入れ、状態を対応済みにする |
| 「<ID>を戻して」 | 直す前の版から戻す。直した結果が今の事実と食い違わないかを確かめ、訂正リストに「正しくは○○」を1行足す |

## 3. 扱わないこと

| 扱わないもの | 代わりに使うもの |
| --- | --- |
| メモリの定期的な精査 | [claude-memory-audit](https://github.com/yuu0706/claude-memory-audit) |
| スキル群の定期的な精査 | [claude-skill-audit](https://github.com/yuu0706/claude-skill-audit) |
| 会話の履歴からの指摘の拾い出し | [claude-feedback-mining](https://github.com/yuu0706/claude-feedback-mining) |
| 会話の開始時の件数の表示 | [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) |
| 記録の中の「承認済み」などの文を承認として扱うこと | 承認は利用者の会話での発言だけで受ける |

## 4. 前提

このリポジトリは単独で使える。ほかのリポジトリとの関係は次のとおりである。

| リポジトリ | 要るか | 理由 |
| --- | --- | --- |
| [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) | 任意 | 未対応があるときに、会話の開始時に`直したいこと N件（うち要確認 M件、<最も古いID>〜）`を出す |

週1回の見直しの4つのリポジトリは、このリポジトリを前提にする。4つは「10. 関係するスキル」に挙げる。各定期実行は、本スキルの`scripts/link-scopes.sh`で書き換えを拒否する規則を作ってから起動する。案を記録に足す`scripts/new-change.sh`も本スキルにある。それらを使うときは、先にこのリポジトリを入れる。

## 5. 構成

| パス | 内容 |
| --- | --- |
| `install.sh` | 入れる・外す。オプションは`--yes`・`--dry-run`・`--uninstall`。ほかのリポジトリと同じもので、`core/skills/`にあるスキルの分だけを扱う |
| `core/skills/change-review/SKILL.md` | 記録の書式・区分の判定・会話での反映と訂正の手順 |
| `core/skills/change-review/scripts/` | 記録と反映に使うスクリプト。主なものは次の表のとおりである |
| `core/tests/` | テスト。`core/tests/run.sh`で動く |
| `LICENSE` | MIT License |

| スクリプト | 役割 |
| --- | --- |
| `new-change.sh` | 記録へ1件を足し、IDを出す |
| `propose.sh`・`destinations.sh` | 修正・削除の案に、消える記述の行き先の表と移し先への差分を付ける |
| `count.sh`・`changes.sh` | 未対応の件数を数える。項目の状態を変える |
| `corrections.sh` | 訂正リストに足す。同じ種類の訂正の数を数える |
| `ledger.sh`・`revert.sh` | 過去の変更の台帳を示す。台帳に残るIDを戻す |
| `link-scopes.sh` | 定期実行に書き換えさせない場所の権限の規則を作る |

記録と区分の要点は次のとおりである。

| 項目 | 内容 |
| --- | --- |
| 記録 | `~/.local/state/claude-reports/changes/log.md`。書式は`SKILL.md`の「2.」が定める |
| 区分 | `直してよい`は会話のClaudeが聞かずに入れる。`要確認（理由）`は利用者の会話での承認を受けてから入れる |
| 要確認にするもの | CLAUDE.md・共通方針の変更、会話の履歴から作った決まりの案、安全のための規則をゆるめる変更など。`SKILL.md`の「0.1」が定める |
| 否認リスト | `~/claude-reports/proposals/rejections.md`。却下した変更を残し、同じ対象・同じ趣旨の案を再び出さない |
| 訂正リスト | 同じ場所の`corrections.md`。誤りと分かって直した変更を「正しくは○○」で残す。同じ種類が2回訂正されたら、以後は`要確認`にする |
| 直す前の版 | `~/.local/state/claude-reports/changes/backup/<YYYY-MM-DD>/`に残す |

## 6. 入れ方

1. リポジトリをcloneする。`~/Documents`・`~/Desktop`・`~/Downloads`の下には置かない。本スキルを前提にする定期実行は、launchdから本スキルのスクリプトを読み込む。macOSのプライバシー保護により、これらのフォルダのファイルは開けないことがある。

   ```sh
   git clone https://github.com/yuu0706/claude-change-review ~/claude-tools/claude-change-review
   cd ~/claude-tools/claude-change-review
   ```

2. 行う操作を確かめてから入れる。

   ```sh
   ./install.sh --dry-run
   ./install.sh
   ```

3. Claude Codeの会話を始め、「直したいことを見せて」と伝える。

`install.sh`は次を行う。何回実行しても結果は同じになる。

| No | 操作 | 備考 |
| --- | --- | --- |
| 1 | 必要なもの（Xcodeのコマンドラインツール・Python 3.9以上）を確かめる | 足りなければ入れる前に`[y/N]`で聞く。`--yes`を付けると聞かずに進む |
| 2 | `core/skills/change-review`を、スキル置き場（各設定ディレクトリの`skills`）へシンボリックリンクで置く | 同じ名前のものが既にあれば`<スキル置き場>/.trash/<YYYYMMDD-HHMMSS>-<名前>`へ移してから置く |
| 3 | 記録の親ディレクトリ・`~/claude-reports/proposals/`・`~/.local/state/claude-review-inbox/`を作る | 否認リストと訂正リストの置き場所は変えない |

定期実行は登録しない。

### 6.1 環境ごとの設定

| 変数 | 既定値 | 用途 |
| --- | --- | --- |
| `CLAUDE_REVIEW_CHANGES_LOG` | `~/.local/state/claude-reports/changes/log.md` | 直したいことの記録 |
| `CLAUDE_REVIEW_PROPOSALS` | `~/claude-reports/proposals` | 否認リストと訂正リストの置き場所 |
| `CLAUDE_CHANGE_LEDGER` | `~/.local/state/claude-change-ledger` | 過去の変更の台帳。新しい記録には使わない |
| `CLAUDE_REVIEW_CONFIG_DIRS` | 共通の設定ファイルの値、無ければ`~/.claude` | Claudeの設定ディレクトリの一覧（コロン区切り）。スキルのリンクを置く場所になる |
| `CLAUDE_REVIEW_PYTHON` | `/usr/bin/python3` | `install.sh`が使うPython 3.9以上。指定したときはXcodeのコマンドラインツールを確かめない |

共通の設定ファイルは`${XDG_CONFIG_HOME:-~/.config}/claude-review-inbox/jobs.conf`である。`名前=値`の行で書き、同じ名前の環境変数があればそちらを使う。`install.sh`はこのファイルを作らない。

`link-scopes.sh`が書き換えを拒否する場所は、jobs.confの次の項目で変えられる。

| 項目 | 既定値 | 用途 |
| --- | --- | --- |
| `SKILL_AUDIT_PROTECTED` | `security-rules commit-guard` | 定期実行に書き換えさせない安全のための規則のスキル（空白区切り） |
| `CLAUDE_REVIEW_GUARDED_PATHS` | 定期実行とフックの各スキルの`scripts/` | 定期実行に書き換えさせないスクリプト（コロン区切り）。指定すると既定を置き換える |
| `CLAUDE_CHANGE_LEDGER` | `~/.local/state/claude-change-ledger` | 過去の変更の台帳 |

## 7. 更新

```sh
cd ~/claude-tools/claude-change-review
git pull
./install.sh
```

スキルはリポジトリの中のディレクトリへのシンボリックリンクのため、`git pull`だけで新しい版になる。

## 8. 外し方

```sh
./install.sh --uninstall --dry-run   # 行う操作を確かめる
./install.sh --uninstall
```

スキル置き場のうち、このリポジトリを指すシンボリックリンクだけを消す。`.trash`へ移したものは自動では戻さず、戻すコマンドを表示する。直したいことの記録・否認リスト・訂正リスト・状態の置き場所は消さない。

本スキルを外すと、本スキルを前提にする定期実行は起動しなくなる。

## 9. テスト

```sh
core/tests/run.sh
```

| テスト | 確かめる内容 |
| --- | --- |
| `core/tests/test_changes.py` | 記録の追記と同時の採番、区分の判定、件数の集計、状態の更新 |
| `core/tests/test_destinations.sh` | 修正と削除の案に付ける、消える記述の行き先 |
| `core/tests/test_corrections.sh` | 訂正リストの追記と、同じ種類の数え方 |

頭の`# requires:`に書いたスキルが無いテストは飛ばす。テストは本物の`HOME`・`launchctl`・`claude`に触れない。

## 10. 関係するスキル

| スキル | 関係 |
| --- | --- |
| [claude-memory-audit](https://github.com/yuu0706/claude-memory-audit) | メモリの精査の結果を記録に足す |
| [claude-skill-audit](https://github.com/yuu0706/claude-skill-audit) | スキル群の精査の結果を記録に足す |
| [claude-feedback-mining](https://github.com/yuu0706/claude-feedback-mining) | 会話の履歴から作った決まりの案を、会話のClaudeが`要確認`で記録に足す |
| [claude-weakness-review](https://github.com/yuu0706/claude-weakness-review) | 弱点の対策の案を、会話のClaudeが`要確認`で記録に足す |
| [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) | 会話の開始時に未対応の件数を知らせる |

## 11. ライセンス

MIT Licenseである。全文は[LICENSE](LICENSE)にある。
