## \_index.scss の使い方

Sass（SCSS）における \_index.scss は、主にフォルダ内の複数のファイルを 1 つにまとめ、外部から呼び出しやすくするために使用されます。

- 基本的な役割  
  フォルダ内に \_index.scss を配置すると、他のファイルからそのフォルダを読み込む際、ファイル名を省略してフォルダ名だけでインポートできます。
- 基本の書き方（@forward）  
  現在の Sass では @import ではなく @forward を使うのが標準的です。

### 例

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

### 書き方のポイント

1. 名前空間の活用  
   @use '../global' と書くと、デフォルトでフォルダ名（global）が名前空間になります。  
   これにより、どのファイルの変数を使っているのかが明確になり、多人数開発でもバグを防げます。  
   また、global ディレクトリに g というエイリアスをつけたら g.$○○ みたいに呼び出すことができます。  
   \_index.scss でまとめる場合、名前空間は global ディレクトリで一つになります。
2. 同じファイルを二度読みしても大丈夫  
   旧来の @import と異なり、@use や @forward は同じファイルを何度読み込んでも、コンパイル後の CSS で重複して出力されることはありません。
3. プライベート変数の保護: 変数名の先頭に \_ または - を付ける（例: $-internal-var）と、@forward しても外部からはアクセスできない「そのファイル専用の変数」にできます。
