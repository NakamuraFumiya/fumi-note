---
title: チーム開発におけるGit操作手順
date: "2020-08-21T22:40:32.169Z"
description: 
---

業務でGitの扱いを学んだのでまとめました！


１. リモートリポジトリからローカルリポジトリへpull

```
git pull origin master
```

２. masterから、新しいブランチを作成

```
git checkout -b ブランチ名
```

３. 作成したブランチをリモートへ反映

```
git push origin ブランチ名
```

４. 修正したファイルをステージングへ移動

```
git add  -A
```

５. 修正したファイルを確認

```
git status
```

６. 修正したファイルが問題なければコミット

```
git commit -m “メッセージ” 
```

７. リモートリポジトリへpush

```
git push origin ブランチ名
```

