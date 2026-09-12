# close-out-retro

Claude Codeのスキル。まとまった作業の終わりに一度呼び出して使う。以下の3つをやる。

1. セッション中に出てきた学びを、Claudeの記憶かプロジェクトのドキュメント（`CLAUDE.md`等）のどちらかに振り分ける
2. 作業中に見つかったが直していないバグ・課題をissueトラッカーと突き合わせて、起票を提案する
3. 同じ手順を何度も手でやっていた場合など、実際にシグナルがあったときだけ新しいスキル/エージェントの候補を提案する

英語版は[README.md](README.md)。

## 背景

セッションの中で、ユーザーからの指摘や地味なバグの回避策、直していない小さな不具合、何度も繰り返した手順などが出てくるが、セッションが終わると忘れられがちだと思う。close-out-retroは、それを作業の区切りごとに拾うためのスキル。

フルの監査ではなく、自動発火のフックでもない。「これで一区切り」というタイミングで呼び出して使う。

## やること

### 1. 学びの振り分け

残しておく価値のあるものについて、それが「このユーザー・プロジェクトとの協働の仕方」に関する話なら記憶へ、「コードやシステムそのもの」の話ならプロジェクトのドキュメント（`CLAUDE.md`、`docs/`等）へ振り分ける。

ドキュメントに書いた内容を記憶側に古いまま重複させないようにする、という点も入れている。

### 2. 未着手issueの棚卸し

セッション中に見つかったが直していないバグ・デッドコード・エッジケースを、issueトラッカーと突き合わせて確認する。新規で確認が取れたものだけ起票を提案する。

確信の持てないものを確定した不具合として書かないようにしている。「たぶんバグかもしれない」くらいのものは、そう伝えるだけにして起票するかはユーザーに任せる。

### 3. スキル/エージェントの提案

以下のようなシグナルがあったときだけ、候補を提案する（作成はしない）。

- 同じ手順を2回以上手で繰り返した
- ユーザーが「毎回面倒」と言った
- 既存のスキルが本来当てはまるはずなのに発火しなかった

毎回「スキルを作りますか?」と聞くようなことはしない。

詳しい仕様は[SKILL.md](SKILL.md)を参照。

## インストール

グローバル:

```bash
mkdir -p ~/.claude/skills/close-out-retro
curl -o ~/.claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

プロジェクトスコープ（このリポジトリだけ）:

```bash
mkdir -p .claude/skills/close-out-retro
curl -o .claude/skills/close-out-retro/SKILL.md \
  https://raw.githubusercontent.com/ryo823/close-out-retro/main/SKILL.md
```

## 使い方

`/close-out-retro`で呼び出すか、「これで締めよう」「残しておくべき学びある?」のように話しかける。

### 実行例（架空のプロジェクト）

架空のAPI `acme-widgets` にレート制限を追加した後、というシナリオ。

```
> /close-out-retro

## 1. 学び
- [記憶] レート制限の設定は環境変数ではなく`config/limits.yml`に置くのが好み、との指摘。
  理由は運用チームが再デプロイ無しで制限値を変更できるようにするため。
- [ドキュメント → CLAUDE.md] ワーカーの再起動順序が重要。設定パッチを当ててから
  `worker restart`する必要がある。逆順だと1サイクルの間だけ古い制限値が残る。

## 2. 未着手issue
- `test/quota_test.py::test_burst`が1回失敗。今回の変更とは無関係で、タイミング起因の
  不安定テストに見えるが未確認。起票はせず判断はお任せする。

## 3. スキル/エージェントの兆候
- 「クォータ確認→設定パッチ→ワーカー再起動→検証→リリースタグ付け」を3回手動で繰り返した。
  `acme-widgets`固有のツールに依存するので、プロジェクトスコープのスキル`quota-rollout`を
  提案する。
```

架空の例で、実プロジェクトのデータは使っていない。

## 関連事例

似たことをやっているものは他にもある。

- [netresearch/retro-skill](https://github.com/netresearch/retro-skill): セッションの学びを記憶/プロジェクトルール/スキルPRに振り分ける`/retro`コマンド。1番目にかなり近い。issueの棚卸しは無い。
- [melodykoh/learning-loop-skill](https://github.com/melodykoh/learning-loop-skill): `CLAUDE.md`/`MEMORY.md`/「Judgment Ledger」に振り分けるscan/wrap-up構成。こちらも1番目に近い。
- [echolimitless氏のQiita記事](https://qiita.com/echolimitless/items/949070036dba433c69a9): セッションのログからパターンを抽出し、`CLAUDE.md`や新規スキルへ昇格させる3層の記憶アーキテクチャ。1番目とかなり重なる。

1番目（学びの振り分け）自体は新しくないと思う。調べた範囲で見当たらなかったのは2番目（issueトラッカー側からの棚卸し）だった。close-out-retroはこの2番目と、提案止まりの3番目を組み合わせたもの。

## ライセンス

[MIT](LICENSE)
