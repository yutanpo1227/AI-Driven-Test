# Codexを用いた自動開発

## Issueの発行をトリガーにIssueを解決させる

1. Issueの発行でGithubActionsをトリガー
2. codexにissueの内容を渡す
3. codexが修正
4. codexがPRを作成
5. PRの作成をトリガーにPRのdiffの内容を要約させる
6. 人間によるレビューとマージ

## mainへのマージをトリガーに自動でIssueを作成する

1. mainへのマージでGithubActionsをトリガー
2. codexにmainのコードを見てもらい修正点を挙げてもらう
3. それぞれIssueを作成してもらう

この2つを繋げることで自動的に実装→修正のループができるはず？

**人間は新たなIssueを投げるだけが理想**