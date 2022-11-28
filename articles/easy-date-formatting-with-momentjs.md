---
title: "MomentJSで日付を簡単にフォーマットする"
emoji: "⏰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript, typescript, momentjs]
published: true
---

# 注意

今回紹介するライブラリは**すでに開発が終了**しています。
後継ライブラリの[luxon](https://moment.github.io/luxon)の使用をご検討ください。

# 本日のお題

**YYYY-MM-DD**のように、日付の表記のフォーマットを定める記法を見かけたことはありますでしょうか？
これは[ISO8601](https://ja.wikipedia.org/wiki/ISO_8601)で定められている国際規格に乗っ取った表記法です。
今回、ユーザーが指定したフォーマットに沿って日付を返す必要があり、その際に利用したMomentJSというライブラリについてご紹介したいと思います。

[レポジトリ: https://github.com/moment/moment](https://github.com/moment/moment)
[ドキュメント: https://momentjs.com](https://momentjs.com)

# MomentJSに触れてみる

MomentJSではMomentオブジェクトを利用することで日付や時間の処理をすることができます。

```ts
const nowMoment = moment(); 
// Moment<2022-11-28T22:48:28+09:00>
```

```ts
const thenMoment = moment("2022-11-28"); 
// Moment<2022-11-28T00:00:00+09:00>
```

また、MomentオブジェクトはDateオブジェクトをラップしているようです。
そのため、DateオブジェクトからMomentオブジェクトを生成したり、その逆を行うことが可能です。

```ts
const nowMoment = moment(new Date("2022-11-28")); 
// Moment<2022-11-28T00:00:00+09:00>
```

```ts
const nowDate = moment().toDate();　
// 2022-11-28T13:50:40.243Z
```

# フォーマット機能が強力

取得した日付をフォーマットするには、Formatメソッドを利用します。

```ts
const now = moment().format("YYYY-MM-DD"); 
// 2022-11-28
```

```ts
const now = moment().format("YY/MM/DD"); 
// 22/11/28
```

```ts
const now = moment().format("MM月DD日"); 
// 221128
```

```ts
const now = moment().format("dddd"); 
// Monday
```

# 日付の差分・加算・減算機能

2000年は何年前だったかな...と思いを馳せたり、

```ts
const ago = moment("20000101", "YYYYMMDD").fromNow();
// 23 years ago
```

1000日前は何月何日だったかな...とよく分からない問いをしたり、

```ts
const ago = moment().subtract(1000, "days").calendar();
// 03/03/2020
```

10年後の自分は何年にいるんだろう...と小泉節を効かてみたり、

```ts
const future = moment().add(10, "years").calendar();
// 11/28/2032
```

1年って何日だったかな...という当たり前の真実を再発見することもできます。

```ts
const fromMoment = moment('2022-01-01 0:00');
const toMoment   = moment('2023-01-01 0:00');
toMoment.diff(fromMoment, 'days')
// 365@[tweet](https://twitter.com/see2et/status/1434468966408146948?s=20&t=yEVIfx40pPdziCNSL-2TPw)

```

# ロケール機能があるらしい

裁判から逃げフランスに引っ越した時なんかのために、ロケールを変更することが可能です。
私の環境では日本に設定しても英語のままだったので残念。
ただ、[自分でカスタマイズ](https://momentjs.com/docs/#/i18n/changing-locale/)することもできるようで...?

```ts
moment.locale('fr');
moment(1316116057189).fromNow(); 
// il y a une heure
```

# 最後に

自分で実装しなくても国際規格に則った細かい記法が反映できるのは素晴らしいですね！
また、本来の目的である日付のフォーマット以外にもたくさんの機能があり、面白かったです。
よろしければ[Twitterアカウント](https://twitter.com/see2et)のフォロー、よろしくお願いします！
@[tweet](https://twitter.com/see2et/status/1434468966408146948?s=20&t=yEVIfx40pPdziCNSL-2TPw)
