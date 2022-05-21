---
title: "ポートフォリオを新調しました"
id: "new-portfolio"
emoji: "✨"
date: "2022-05-21"
tags: ["develope", "new"]
---

## 目次

## TL;DR

ポートフォリオを作り直した。

1. 高速
2. 拡張性
3. ナウい

## 高速

フレームワークに[Next.js](https://nextjs.org)を採用してSGを行っている。

~~Vercelの犬~~になるのは悔しいが、我慢しよう。手軽にfontと画像の最適化を行ってくれるのは魅力だからだ。

SG とはいいながら、ページ遷移のたびに JS が走るので動きはなめらかだ（言い方を変えれば、ピュアじゃない）。

画像の管理に[Cloudinary](https://cloudinary.com)を用いて最適化された画像を配信している。

画像のサイズは表示速度に直結するのでとても助かる。

## 拡張性

### 全体を通して

コンポーネント駆動開発[^1]と呼ばれるプラクティスに則って[Atomic Design](https://atomicdesign.bradfrost.com)(なんちゃって)なコンポーネント設計を意識したので再利用がしやすく拡張も楽だ。

### Webページ

できるだけ情報を最新に保ちつつ、情報を変える際に[このリポジトリ](https://github.com/re-taro/re-taro.dev/)へ直接commitを残さないため、APIサーバーを建てそこを介して情報をやり取りした。

### ブログ

Markdown(内容管理) + tsx(テンプレートエンジン)。Markdown ならそう簡単には廃れないだろうし、いつか別サービスにも投げ込める安心感があると、ラマヌジャンは言った(知らんけど)。

Markdownの処理系には[unified](https://github.com/unifiedjs/unified)資産を用いた。

様々なunifiedのAPIを組み合わせて構築するから楽しかった。

## ナウい

ナウい。そうだ、僕はナウいのが好きだ。下は僕が弊クラブのLTで話した際の資料を抜粋したもので、このポートフォリオに使われている技術の一覧である。

![ナウくね？](https://res.cloudinary.com/re-taro/image/upload/q_60/f_auto/v1653117372/posts/new-portfolio/new-portfolio1_ezd66i.png)

### フロントエンド

フレームワークにNext.js、スタイリングに[Tailwind CSS](https://tailwindcss.com)(ただしCSS in JSで使いたかったから今回は[twin.macro](https://github.com/ben-rogerson/twin.macro)を使用)。

GraphQLクライアントにUrqlを使っている。

ホスティング先はVercelだ(~~犬です、ワンワン~~)。

https://tailwindcss.com

https://github.com/ben-rogerson/twin.macro

### バックエンド

Juniperを使っている。お気づきだろうか？[Rust](https://www.rust-lang.org)でGraphQLサーバーを書いてみている。

これがまた難しい。すぐにでも[Nest.js](https://nestjs.com)あたりに逃げたい。

デプロイ先は[render](https://render.com)に任せっきりだ(Heroku？あいつは...)。

作ってて思ったがGraphQLは正義だ。

https://www.rust-lang.org

https://nestjs.com

### その他

アナリティクスに今回は[umami](https://umami.is)を使ってみている。めっちゃUIかわいくてお気に入りだ。

https://umami.is

## 機能

以下では具体的に実装できた機能と使ったライブラリなどを書いていく。

### Tailwind CSS & twin.macro

Tailwind CSSは正義だとは僕は思わない。だが解の一つではあると思う。

また今回Atomic Designを採用した関係で、CSS in JSでの開発が最適[^2]であると思い、`twin.macro`を用いた。

twin.macroには一つ大きな問題点があった。それはTailwind CSS v3に対応していないということだ。

https://github.com/ben-rogerson/twin.macro/issues/589

↑一応issueは立っているため経過観察だ。

Tailwind CSS v3から追加された`flex-basis`などは当然サポートされていないため下のような処置を取った。

![ゴリ押しは正義](https://res.cloudinary.com/re-taro/image/upload/q_60/f_auto/v1653123753/posts/new-portfolio/new-porfolio2_n4kgtd.png)

### Urql

GraphQLのクライアントライブラリというと何を思いつくだろう。有名どころだとApolloやRelayがある。話が変わるが`@next/bundle-analyzer`というライブラリがある。これでビルドして吐き出されたバンドルサイズを見てみよう。

きっと驚くだろう、ApolloやRelayのサイズの大きさを。それに比べてUrqlのサイズの小ささを。気になる人は下でUrqlが比較を行っている。

https://formidable.com/open-source/urql/docs/comparison/

もちろん、適材適所ではある。今回はqueryだけでよかったのでなるべく小さいものにした。

```ts
/* pages/index.tsx */

// 略

// eslint-disable-next-line unicorn/prevent-abbreviations
export const getStaticProps: GetStaticProps<{ meta: SeoProperties; urqlState: SSRData }> = async () => {
  const client = await urqlClient();
  await client.query(HomeDocument).toPromise();
  const meta: SeoProperties = {
    description: "Rintaro Itokawa's Dev Site | re-taro",
    ogImageUrl: encodeURI(`${OGP_HOST}/api/ogp?title=re-taro`),
    pageRelPath: "",
    pagetype: "website",
    sitename: "re-taro.dev",
    title: "Rintaro Itokawa - Emotion Seeker",
    twcardtype: "summary_large_image",
  };
  return {
    props: {
      meta,
      urqlState: ssrCache.extractData(),
    },
  };
};

const HomePage: NextPage<Properties> = ({ meta }) => {
  const [response] = useQuery<HomeQuery>({ query: HomeDocument });
  return <Home data={response.data} meta={meta} />;
};

export default withUrqlClient(
  () => ({
    url: END_POINT,
  }),
  { neverSuspend: true, ssr: false },
)(HomePage);
```

少し変わった使い方をするがとてもシンプルで使いやすかった。

### GitHub Flavored Markdown

`remark-gfm`で対応した。

https://github.com/remarkjs/remark-gfm

```md
| 表を     | 作る       |
| -------- | ---------- |
| たとえば | このように |
| 要素を   | 増やす     |

https://re-taro.dev

みたいな生のリンクも置けるし

- こうやって
  - リストが書ける。さらに、[^3]
```

| 表を     | 作る       |
| -------- | ---------- |
| たとえば | このように |
| 要素を   | 増やす     |

https://re-taro.dev

みたいな生のリンクも置けるし

- こうやって
  - リストが書ける。さらに、[^3]

### 絵文字

`remark-gemoji`で変換する。

https://github.com/remarkjs/remark-gemoji

`:v:`が:v:になる

### 数式

`remark-math`と`rehype-katex`を噛ませる。

https://github.com/remarkjs/remark-math

https://github.com/remarkjs/remark-math/tree/main/packages/rehype-katex

```tex
> $$
> f(x)=\sum^{\infty}_{k=0}f^{(k)}(0)\frac{x^k}{k!}
> $$
```

> $$
> f(x)=\sum^{\infty}_{k=0}f^{(k)}(0)\frac{x^k}{k!}
> $$

$e^{i\pi}+1=0$のようなインライン数式を記述できる。

スタイルシートを読み込むだけで設定が可能なので楽だ。

```ts
// components/organisms/post-meta/index.tsx

// 略

const PostMeta: React.FC<PostMetaPropeties> = ({ meta }) => (
  <React.Fragment>
    <Seo {...meta} />
    <Head>
      <link
        rel={"stylesheet"}
        href={"https://cdn.jsdelivr.net/npm/katex@0.15.6/dist/katex.min.css"}
        integrity={"sha384-ljao5I1l+8KYFXG7LNEA7DyaFvuvSCmedUf6Y6JI7LJqiu8q5dEivP2nDdFH31V4"}
        crossOrigin={"anonymous"}
      />
      <title></title>
    </Head>
  </React.Fragment>
);

// 略
```

### ルビ

[@haxibami](https://github.com/haxibami)さんが作成した`remark-jaruby`を用いた。

```md
> {水晶機巧－ハリファイバー}^(我らが母)
```

> {水晶機巧－ハリファイバー}^(我らが母)

以上を合わせた`remark-parse` / `remark-rehype`まわりのメソッドチェーンが下の通り

```ts
// utils/parser.ts

import rehypeShiki from "@re-taro/rehype-shiki";
import rehypeAutolinkHeadings from "rehype-autolink-headings";
import rehypeKatex from "rehype-katex";
import rehypeSlug from "rehype-slug";
import rehypeStringify from "rehype-stringify";
import remarkGemoji from "remark-gemoji";
import remarkGfm from "remark-gfm";
import remarkJaruby from "remark-jaruby";
import remarkMath from "remark-math";
import remarkParse from "remark-parse";
import remarkRehype from "remark-rehype";
import remarkToc from "remark-toc";
import remarkUnwrapImages from "remark-unwrap-images";
import * as shiki from "shiki";
import stripMarkdown from "strip-markdown";
import { unified } from "unified";

const MdToHtml = async (md: string) => {
  const result = await unified()
    .use(remarkParse)
    .use(remarkGfm)
    .use(remarkGemoji)
    .use(remarkMath)
    .use(remarkJaruby)
    .use(remarkUnwrapImages)
    .use(remarkToc, {
      heading: "目次",
      tight: true,
    })
    .use(remarkRehype)
    .use(rehypeKatex)
    .use(rehypeShiki, {
      highlighter: await shiki.getHighlighter({ theme: "nord" }),
    })
    .use(rehypeSlug)
    .use(rehypeAutolinkHeadings, {
      behavior: "wrap",
    })
    .use(rehypeStringify)
    .process(md);
  return result.toString();
};

export { MdToHtml }
```

また、`rehype-react`関連の処理は以下のようになる。

```ts
// lib/rehype-react.ts

import React from "react";
import rehypeParse from "rehype-parse";
import rehypeReact from "rehype-react";
import type { Options as RehypeReactOptions } from "rehype-react";
import { unified } from "unified";
import { Image } from "~/components/molecules/image";
import type { ImageProperties } from "~/components/molecules/image";
import { Link } from "~/components/molecules/link";
import type { LinkProperties } from "~/components/molecules/link";

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const RehypeReact = (html: string): React.ReactElement<unknown, string | React.JSXElementConstructor<any>> => {
  const result = unified()
    .use(rehypeParse, {
      fragment: true,
    })
    .use(rehypeReact, {
      components: {
        // eslint-disable-next-line id-length
        a: (properties: LinkProperties) => Link(properties),
        img: (properties: ImageProperties) => Image(properties),
      },
      createElement: React.createElement,
    } as RehypeReactOptions)
    .processSync(html);
  return result.result;
};

export { RehypeReact };
```

以上で非常に書きやすいブログになった。

### 動的OGP

[@haxibami](https://github.com/haxibami)さんの実装を見つつ、Vercelにデプロイした。

https://github.com/re-taro/ogp.re-taro.dev

![OGP](https://res.cloudinary.com/re-taro/image/upload/q_60/f_auto/v1653141535/posts/new-portfolio/new-porfolio3_pdt0ou.png)

### 自動再デプロイ

[データを置いているリポジトリ](https://github.com/re-taro/re-taro.d)が更新されるとactionsが発火して[ポートフォリオのリポジトリ](https://github.com/re-taro/re-taro.dev)へdispatchを送りそれを受け取った方で自動ビルドが走る。なかなかシャレオツで気に入ってる。

ディスパッチを送る側
```yaml
name: Dispatch
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: |
          curl -vv -H "Authorization: token ${{ secrets.DISPATCH_TOKEN }}" -H "Accept: application/vnd.github.everest-preview+json" "https://api.github.com/repos/re-taro/re-taro.dev/dispatches" -d '{"event_type": "update"}'
```

ディスパッチを受け取る側
```yaml
name: Dispatch production build

on:
  repository_dispatch:
    types: [update]

jobs:
  // 実行したいもの
```

## 感想

めちゃくちゃいい仕上がりになったと自負している。


[^1]: https://qiita.com/UCLab1421/items/1c4e4acfdc785dbfa269
[^2]: https://zenn.dev/t_keshi/articles/emotional-usage-of-emotion
[^3]: 脚注も使える
