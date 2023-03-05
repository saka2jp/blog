---
title: "Python3への移行リスクを最小限にする"
emoji: "🐍"
type: "tech"
topics: [Python, 勉強会, Programming]
published: true
published_at: 2020-02-28 02:00
---

Python2 EOL party in Tokyo で「Python3 への移行リスクを最小限にする」というタイトルで登壇させていただきました。

https://python2.connpass.com/event/161403/

運営のみなさま、登壇機会をいただきましてありがとうございました。

資料は SpeakerDeck で公開しています。

https://speakerdeck.com/saka2jp/how-to-reduce-risk-with-upgrading-to-python3

Twitter での反応をまとめました。
つぶやいていただいたみなさま、ありがとうございました。

https://togetter.com/li/1468904

以下に Markdown 形式の資料も公開します。

# Python3 への移行リスクを最小限にする

## 今回のテーマ

「移行リスクを最小限に留める」

コストをかければリスクは減らせる

Python3 への移行のために無限にコストをかけられるわけではない
機能のリリースやバグフィックスが優先され、むしろほとんどコストをかけられない

割り振られたリソースの中でリスクを可能な限り減らすのがエンジニアの役目
そのために何ができるのか

## Why We Need to Update?

1.  脆弱性
    - 新たなセキュリティバグが発覚した場合に、攻撃を受けるリスクがある
2.  新機能
    - コミュニティの努力の恩恵を受けることができる
3.  脱レガシー
    - 誰しも好んでレガシーな環境を選ばない
    - 新しい人は入ってこないし、既存のメンバーも離脱してしまうかも

## How to Upgrade 2 to 3

### 1. 自動変換ツール

- [2to3](https://docs.python.org/3/library/2to3.html)
  - Python2 を Python3 のコードに完全に変換
- [Futurize](https://python-future.org/futurize.html) / [Modernize](https://python-modernize.readthedocs.io/en/latest/)
  - Python2/3 で動作するコードを生成

**注意点**

- すべて自動化されているわけではない
  - 手動でコード修正が必要な箇所がある
- Python2.7 サポート
  - Python2.7 までは自力であげる必要がある

### 2. サードパーティのアップデート

Python3 をサポートしているバージョンまであげる
最新のバージョンとの差分を確認 -> 更新

```bash
$ pip list -o
Package  Version  Latest  Type
--------  --------  ------- -----
requests  2.21.0    2.22.0  wheel
urllib3      1.24.3    1.25.8   wheel

$ pip install -U requests
```

**注意点**

- 後方互換性がない変更が含まれる可能性
  - ドキュメントを確認しながら
- そもそも Python3 をサポートしていない可能性
  - 独自で置き換える
  - フォークしたプロジェクトを探す

### pyup.io

GUI で最新バージョンとの差分を確認できる
CHANGELOG があれば表示してくれる

https://pyup.io/tools/requirements-checker/

## What We Should to Reduce Risk

リスクを減らすために何ができるか

### 1. Python2 と Python3 の違いを理解する

どんなリスクがあるのか（互換性がない箇所）を理解する
リスクのある箇所を重点的に対策する
Python 公式ドキュメントによれば警戒するべき違いは 2 つだけ

#### (1) 文字列の扱い

Python2 での encode/decode は unicode 型と str 型
Python3 での encode/decode は str 型と byte 型
Python3 の str 型は Python2 の str 型ではなく unicode 型と同等

```python
$ python2
>>> u'ぱいそん'.encode('utf-8')
'\xe3\x81\xb1\xe3\x81\x84\xe3\x81\x9d\xe3\x82\x93'
>>> '\xe3\x81\xb1\xe3\x81\x84\xe3\x81\x9d\xe3\x82\x93'.decode('utf-8')
u'\u3071\u3044\u305d\u3093'
```

```python
$ python3
>>> 'ぱいそん'.encode()
b'\xe3\x81\xb1\xe3\x81\x84\xe3\x81\x9d\xe3\x82\x93'
>>> b'\xe3\x81\xb1\xe3\x81\x84\xe3\x81\x9d\xe3\x82\x93'.decode()
'ぱいそん'
```

#### (2) 除算

Python3 では int 同士の除算結果はすべて float になる

```bash
$ python2
>>> 1 / 2
0
```

```bash
$ python3
>>> 1 / 2
0.5
```

### 2. テスト

移行前と移行後の動作が同じであることを担保する
ざまざまなスコープがある

- ユニットテスト：関数やメソッドレベルで
- 結合テスト：2 以上のコンポーネントを組み合わせて
- E2E テスト：サービス全体を通して

#### Testing Pyramid

どのテストをどれだけ実施するのがよいのか
Google が語る最適なテスト戦略
Python3 への移行で最も費用対効果が高いのはユニットテスト

https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html

#### Python のユニットテスト

より高いカバレッジが望ましいがコストが高い
後方互換性がない箇所（文字列や除算）に特に気をつけてテストをかくとよい

#### ユニットテストの書き方

先人たちの心強い参考文献

- [効果的な unittest - または、callFUT の秘密](http://pelican.aodag.jp/xiao-guo-de-naunittest-mataha-callfutnomi-mi.html)
- [できる!Django でテスト!](https://tell-k.github.io/djangocongressjp2018)
- テスト駆動 Python

PyCon JP 2019 ではテストに関するトークが豊富にあった
最新の情報として補填するとよい

- [pytest による CI レボリューション](https://speakerdeck.com/abenben/pytestniyorucireboriyusiyon)
- [unittest で始めるユニットテスト入門](https://docs.google.com/presentation/d/1F0fkcbuuQBCbVDLD2v8YQlMOqEMnF_PpI2AAR5nwhfI)
- [Python を使った API サーバー開発を始める際に整備した CI とテスト機構](https://speakerdeck.com/hgsgtk/python-api-ci-test)

### 3. 継続的インテグレーション

ユニットテストの実行やカバレッジの計測を自動化し継続的に実行する
カバレッジは常に可視化できるようにする

ユニットテストはそのときの実装者がそのときに書くのが最もコストが低い
誰かがテストを追加してくれる保証なんてない
ユニットテストを決して後回しにしない

テストを書くメリットを共有し、テストやカバレッジを CI に組み込み仕組み化することで、先にテストを書く文化を作る。

### 4. サードパーティライブラリの選定

移行の中でも一つの障壁になりうる
依存しているサードパーティの数が多いほど移行のリスクが高くコストも高い
サードパーティを導入する意思決定は慎重に

- 標準ライブラリでは解決できないのか
- サードパーティでしか解決できないのか

それでも導入する場合は以下のような指標からどのサードパーティがよいのか、今後もメンテナンスされ続ける見込みはあるのか判断する

- 直近のリリースはいつか
- どれくらいの頻度でリリースされているか
- コミッターやコントリビューターの人数は何人か
- どれくらい利用されているのか（[PePy](https://pepy.tech/)）
- スター数

### 5. サードパーティのアップグレード自動化

- バージョンアップの Pull を自動で作成してくれるツールを利用する
- デベロッパーはテストの CI が構築できていれば CI がパスしたら基本的にはマージすればよいだけ
- メジャーバージョンのアップグレードはテストがパスしていても気をつける

  - 一気にバージョンをあげるのはリスクが高い
  - 心理的な負担も大きいので誰もやりたくない -> さらに放置される という悪循環になりかねない
  - 日々コツコツと上げつづけることが重要

- Dependabot
  - 2019 年に GitHub に買収された Automated dependency updates サービス
  - Pipenv, Poetry, requirements.txt をサポート

https://dependabot.com/

- Renovate
  - プラットフォーに依存しない
  - GitLab, Bitbacket もサポート

https://renovate.whitesourcesoftware.com/

## Python3 での教訓

バージョンアップをスポットの作業と捉えない
継続的に取り組みを行うことで移行のリスクもコストも下げる

## 参考文献

- [Porting Python 2 Code to Python 3](https://docs.python.org/3/howto/pyporting.html)
- [Automatic conversion to Py2/3](http://python-future.org/automatic_conversion.html)
