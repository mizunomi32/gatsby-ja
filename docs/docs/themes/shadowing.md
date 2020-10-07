---
title: Gatsbyのテーマにおけるシャドウイング
---

Gatsbyのテーマにはシャドウイングの概念が導入されています。この機能を使うことで、webpackバンドル内の`src`ディレクトリに含まれるファイルを独自の実装に置き換えが可能となります。 これは、Reactのコンポーネントや`src/pages`のページ、JSONファイル、TypeScriptファイル、他のインポートされたファイル（`.css`など）に対して使うことができます。

実用的な使用例として、`gatsby-theme-blog`をインストールし、著者の`Bio`コンポーネントをカスタマイズして独自の経歴コンテンツを追加したい場合を取り上げます。シャドウイングを使うと、テーマの元のファイル`gatsby-theme-blog/src/components/bio.js`を独自のファイルに置き換えて、必要な変更を加えることができます。

## シャドウイングの例

`gatsby-theme-blog`をインストールした場合は、`BlogPost`テンプレートで使われる`Bio`コンポーネントがレンダリングされることに気付くと思います。この`Bio`コンポーネントを変更したいときは、シャドウイングAPIを使うことで変更できます。

### テーマファイルの構造

`gatsby-theme-blog`のファイル構造を調べて、シャドウイングをするファイルのファイルパスを見つけることができます。

```text:title="gatsby-theme-blogの構造"
├── gatsby-config.js
├── gatsby-node.js
└── src
    ├── components
    │   ├── bio-content.js
    │   ├── bio.js // highlight-line
    │   ├── header.js
    │   ├── headings.js
    │   ├── home-footer.js
    │   ├── layout.js
    │   ├── post-footer.js
    │   ├── post.js
    │   ├── posts.js
    │   ├── seo.js
    │   └── switch.js
    ├── gatsby-plugin-theme-ui
    │   ├── colors.js
    │   ├── components.js
    │   ├── index.js
    │   ├── prism.js
    │   ├── styles.js
    │   └── typography.js
    └── templates
        ├── post.js
        └── posts.js
```

### `Bio`コンポーネントのカスタマイズ

ここでは`gatsby-theme-blog/src/components/bio.js`をシャドウイングするケースを取りあげます。

シャドウイングAPIは、決定的なファイル構造を使用して、レンダリングされるコンポーネントを決定します。そこで、`gatsby-theme-blog`の`Bio`コンポーネントを上書きするには、`user-site/src/gatsby-theme-blog/components/bio.js`という名前のファイルを作成します。

テーマの`src`ディレクトリである`gatsby-theme-blog/src`ディレクトリにある同じ名前のファイルのかわり、ユーザーのサイトの`src/gatsby-theme-blog`ディレクトリにあるファイルが使われます。
これによって、該当のファイル全体が置き換えられます。
機能やスタイルなど、テーマの元のファイルの一部を再利用したいときは、シャドウファイルの[拡張](#extending-shadowed-files)と[インポート](#importing-the-shadowed-component)に関するこのドキュメントのセクションを確認してください。

ここでは、`gatsby-theme-blog/src/components/bio.js`の代わりに`user-site/src/gatsby-theme-blog/components/bio.js`がレンダリングされます。

```jsx:title=src/gatsby-theme-blog/components/bio.js
import React from "react"
export default () => <h1>My new bio component!</h1>
```

Bioコンポーネントのシャドウイングをおこなうと、次のディレクトリツリーが作成されます。

```text
user-site
└── src
    └── gatsby-theme-blog
        └── components
            └── bio.js // highlight-line
```

## 他のテーマに対してシャドウイングする

`gatsby-theme-blog`を含むいくつかのテーマは、内部で他のテーマを利用しています。たとえば、`gatsby-theme-blog`では`gatsby-plugin-theme-ui`を使用します。このテーマの実装をカスタマイズしたい場合も、シャドウイングを使うことでカスタマイズできます。

たとえば、`gatsby-plugin-theme-ui`から`index.js`をシャドウイングするには、`user-site/src/gatsby-plugin-theme-ui/index.js`という名前のファイルを作成します。

```js:title=src/gatsby-plugin-theme-ui/index.js
import baseTheme from "gatsby-theme-blog/src/gatsby-plugin-theme-ui/index"

export default {
  ...baseTheme,
  fontSizes: [12, 14, 16, 24, 32, 48, 64, 96, 128],
  space: [0, 4, 8, 16, 32, 64, 128, 256],
  colors: {
    ...baseTheme.colors,
    primary: `tomato`,
  },
}
```

このときは、次のディレクトリツリーが作成されます。

```text
user-site
└── src
    └── gatsby-plugin-theme-ui
        └──index.js // highlight-line
```

## 全ファイルに対してシャドウイング可能

シャドウイングAPIはReactコンポーネントのみに制限されていません。`src`ディレクトリにあるJavaScriptファイルやMarkdownファイル、MDXファイル、CSSファイルといったファイルに対しても上書き可能です。これによって、テーマが提供するすべての機能やコンテンツ、スタイルをきめ細かく制御できます。

If you wanted to shadow a CSS file in `gatsby-theme-awesome-css` that's found at `src/styles/bio.css` you can do so by creating `user-site/src/gatsby-theme-awesome-css/styles/bio.css`

```css:title=user-site/src/gatsby-theme-awesome-css/styles/bio.css
.bio {
  border: 10px solid tomato;
}
```

The theme's `bio.css` file would then be replaced with your new CSS file.

## File extensions can be overridden

As long as the theme author imports components/files without the file extension, users are able to shadow these with other types of files. For example the theme author created a TypeScript file at `src/components/bio.tsx` and uses it in another file:

```jsx:title=src/components/header.tsx
import Bio from "./bio"

/* Rest of the code */
```

You'll be able to shadow the Bio file by creating e.g. a JavaScript file at `src/gatsby-theme-blog/components/bio.js` as the file extension wasn't used in the import.

## Extending shadowed files

In addition to overriding files, you can _extend_ shadowable files.

This means that you can import the component you’re shadowing and then render it. Consider a scenario where you have a custom `Card` component that you want to wrap the author’s bio in.

Without extending the component, you would have to manually copy over the entire component implementation from the theme to wrap it with your custom shadowed component. It might look something like:

```jsx:title=src/gatsby-theme-blog/components/bio.js
import React from "react"
import { Avatar, MediaObject, Icon } from "gatsby-theme-blog"
import Card from "../components/card"

export default ({ name, bio, avatar, twitterUrl, githubUrl }) => (
  <Card>
    <MediaObject>
      <Avatar {...avatar} />
      <div>
        <h3>{name}</h3>
        <p>{bio}</p>
        <a href={twitterUrl}>
          <Icon name="twitter" />
        </a>
        <a href={githubUrl}>
          <Icon name="github" />
        </a>
      </div>
    </MediaObject>
  </Card>
)
```

This workflow isn’t too bad, especially since the component is relatively straightforward. However, it could be optimized in scenarios where you want to wrap a component or pass a different prop without having to worry about the component’s internals.

## Importing the shadowed component

In the above example it might be preferable to be able to import the `Bio` component and wrap it with your `Card`. When importing, you can do the following instead:

```jsx:title=src/gatsby-theme-blog/components/bio.js
import React from "react"
import { Author } from "gatsby-theme-blog/src/components/bio"
import Card from "../components/card"

export default props => (
  <Card>
    <Author {...props} />
  </Card>
)
```

This is a quick and efficient way to customize rendering without needing to worry about the implementation details of the component you’re looking to customize. Importing the shadowed component means you can use composition, leveraging a great feature from React.

### Applying new props

In some cases components offer prop APIs to change their behavior. To extend a component you can import it and then add a new prop.

For example, if `NewsletterCTA` accepts a `variant` prop which changes the look and colors of the call to action, you can use it when you extend the component. Below, `NewsletterCTA` is re-exported and `variant="link"` is added in the shadowed file to override its default value.

```jsx:title=src/gatsby-theme-blog/components/newsletter/call-to-action.js
import { NewsletterCTA } from "gatsby-theme-blog/src/components/newsletter"

export default props => <NewsletterCTA {...props} variant="link" />
```

## Using the CSS prop

In addition to passing a different prop to a component you’re extending, you might want to apply CSS using the [Emotion CSS prop](/docs/emotion/). This will allow you to change the styling of a particular component without changing any of its functionality.

```jsx:title=src/gatsby-theme-blog/components/newsletter/call-to-action.js
import { NewsletterCTA } from "gatsby-theme-blog/src/components/newsletter"

export default props => (
  <NewsletterCTA
    css={{
      backgroundColor: "rebeccapurple",
      color: "white",
      boxShadow: "none",
    }}
    {...props}
  />
)
```

**Note:** For this approach to work NewsletterCTA has to accept a `className` property to apply styles after the CSS prop is transformed by the Emotion babel plugin.

```js:title=src/gatsby-plugin-theme-ui/index.js
import theme from "gatsby-plugin-theme-ui/src"

const { colors } = theme
export default {
  ...theme,
  colors: {
    ...colors,
    primary: "tomato",
  },
}
```

This provides a nice interface to extend an object if you want to change a couple values from the defaults.

## How much shadowing is too much shadowing?

If you find yourself shadowing a large number of components in a particular theme, it might make sense to fork and modify the theme instead. The official Gatsby themes support this pattern using a set of `-core` themes. For example, `gatsby-theme-blog` relies on `gatsby-theme-blog-core` so you can fork `gatsby-theme-blog` (or skip it completely) to render your own components without having to worry about dealing with any of the page creation or data sourcing logic.
