# PoC: Dependabot cooldown と セキュリティ更新

「公開後7日は待つ」運用と、PHP2Shellのような即時対応が必要な脆弱性が両立するかを確かめる検証用。

## Part A: uv 側（ローカルで完結・実施済み）

`pyproject.toml` の `exclude-newer = "7 days"` の挙動を、毎日リリースされる boto3 で観察する。
結果は `../references.md` の「検証結果」を参照。

再現:

```powershell
cd poc
uv lock                     # 実験1: 7日より新しい版が除外される
```

## Part B: Dependabot 側（GitHub上の検証用リポジトリが必要・未実施）

案件のリポジトリは使わない。公開してよい検証用の個人リポジトリを新規に作る。

| 実験 | 内容 | 期待する観察 |
|---|---|---|
| B1 | 公開3日以内の版が出ているパッケージを古い版で固定し、version updates を有効にする | PRが出ない / 3日後に出る（cooldown 7日なら7日後） |
| B2 | 既知CVEのある古い版（例: `urllib3<2`系の脆弱版など）で固定し、Dependabot alerts + security updates を有効にする | cooldownに関係なくPRが即時に出る |
| B3 | B2のPRに対し、`exclude-newer = "7 days"` 下でCIの `uv lock` / `uv sync` が通るか | 修正版が公開7日以内なら解決失敗するか（要確認） |
| B4 | `cooldown` に `semver-major-days` 等を指定した時の挙動 | 要確認 |

### 手順メモ

1. 新規 public リポジトリを作成し、この `poc/` の中身をルートに置く
2. Settings > Advanced Security で Dependabot alerts / security updates / version updates を有効にする
3. B2用に、脆弱な古い版を `dependencies` に固定する（`uv lock` はexclude-newerの影響を受けない古い版なら通る）
4. `gh pr list --author "app/dependabot"` でPRの出方を観察し、`gh pr view --json title,body,createdAt` を保存する
5. PRのCI（`uv lock --check` / `uv sync`）の結果を記録する

### 注意

- `cooldown` の対象エコシステム・パラメータは公式ドキュメントで確認すること（未確認）
- `.github/dependabot.yml` の `package-ecosystem: "uv"` が使えるかは検証時に確認する
