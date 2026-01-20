## 今回の課題で心がけたこと

### メンテナンス性の考慮

前回の課題では scss でネストを多用して特異性が上がり、メンテナンスしにくいコードになってしまった。
その反省点を活かし、可能な限りネストは 3 段階までにしメンテナンス性を意識した。

### Figma デザインを反映しやすい CSS 設計

html に**font-size: 62.5%**、body に**font-size: 1.6rem**を指定した。  
body の font-size を 16px にしながら**1rem = 10px**になり計算がしやすくなった。これにより px 単位で書かれている Figma デザインを見て作業がしやすくなった。

### CSS 変数（CSS カスタムプロパティ）の利用

前回の課題ではフォントサイズに「clamp()」が多用されており、すべて入力したところかなりの手間になってしまった。
そこで今回は再利用しそうな font-size を:root に CSS 変数として収めることで値の共通化・メンテナンス性向上に努めた。

## 備忘録

### :root に書く CSS 変数（カスタムプロパティ）

#### :root に変数を書くのは何のため？

:root {
--main-color: #ff7a00;
}

- JS からも触れる
- テーマ切り替えができる
- CSS としてそのままブラウザに残る

#### Sass の$変数との使い分け

Sass の$変数はビルド時に消えて**ただの値になる**ので

- 設計・計算・レイアウト用 → Sass 変数$
- 色・余白・テーマ → CSS 変数--

#### 具体的な使い方

1.  \_root.scss ファイルを作成し以下の例のような内容を書く。

    :root {

    /_ color _/  
     --color-main: #ff7a00;  
     --color-text: #222;  
     --color-bg: #fff;

    /_ font _/

    --font-base: "Noto Sans JP", sans-serif;  
     --fs-base: clamp(1.4rem, 1.2rem + 0.5vw, 1.6rem);

    /_ spacing _/

    --space-xs: 0.4rem;  
     --space-sm: 0.8rem;  
     --space-md: 1.6rem;  
     --space-lg: 3.2rem;  
    }

2.  main.scss で読み込む(@use)

3.  CSS 変数を使う

    .button {

    background-color: var(--color-main);  
     color: var(--color-bg);  
     padding: var(--space-sm) var(--space-md);  
     font-family: var(--font-base);  
    }

### \_index.scss の使い方

Sass（SCSS）における \_index.scss は、主にフォルダ内の複数のファイルを 1 つにまとめ、外部から呼び出しやすくするために使用されます。

- 基本的な役割  
  フォルダ内に \_index.scss を配置すると、他のファイルからそのフォルダを読み込む際、ファイル名を省略してフォルダ名だけでインポートできます。
- 基本の書き方（@forward）  
  現在の Sass では @import ではなく @forward を使うのが標準的です。

#### 例

src/scss/  
 ├── global/  
 │ ├── \_variables.scss  
 │ ├── \_mixins.scss  
 │ └── \_index.scss <-- ① ここで変数を転送  
 └── components/  
　 └── \_button.scss <-- ② ここで読み込んで使う

① global/\_index.scss (変数の集約・公開)
@forward を使って、外部から global フォルダを読み込んだ時に変数を使えるようにします。

// global/\_index.scss  
@forward 'variables';  
@forward 'mixins';

② components/\_button.scss (変数の利用)  
変数を実際に使うファイルでは、@use を使って読み込みます。このとき、パスは \_index.scss があるディレクトリを指定するだけで済みます。

// components/\_button.scss  
@use '../global'; // index.scss を自動的に読み込む

.button {  
 // 「ディレクトリ名.変数名」でアクセスする  
 background-color: global.$primary-color;  
 // 名前空間（global）を省略して使いたい場合は 'as _' を指定  
 // @use '../global' as _;  
 // color: $text-color;  
}

#### 書き方のポイント

1. 名前空間の活用  
   @use '../global' と書くと、デフォルトでフォルダ名（global）が名前空間になります。  
   これにより、どのファイルの変数を使っているのかが明確になり、多人数開発でもバグを防げます。  
   また、global ディレクトリに g というエイリアスをつけたら g.$○○ みたいに呼び出すことができます。  
   \_index.scss でまとめる場合、名前空間は global ディレクトリで一つになります。
2. 同じファイルを二度読みしても大丈夫  
   旧来の @import と異なり、@use や @forward は同じファイルを何度読み込んでも、コンパイル後の CSS で重複して出力されることはありません。
3. プライベート変数の保護: 変数名の先頭に \_ または - を付ける（例: $-internal-var）と、@forward しても外部からはアクセスできない「そのファイル専用の変数」にできます。
