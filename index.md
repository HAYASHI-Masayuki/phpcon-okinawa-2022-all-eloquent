---
title: 30分で理解するEloquentの巨大な全貌
author: HAYASHI Masayuki
---

<style>
h1, p {
  text-align: center;
}
</style>

# 30分で理解するEloquentの巨大な全貌

HAYASHI Masayuki

<!--

TODO: 最初のあいさつどうする？

-->

---

<!-- (起) -->

# Eloquentしっかり理解して使えてますか？

<!--

みなさん、Eloquent, しっかり理解して使えてますか？

Eloquent, 短いコードで、データベースを簡単に扱えて便利ですよね。

簡単なことは簡単にできて、複雑なこともそれなりにできます。

ただ、「本当に」ややこしいことをしようとしたとき、

あるいは逆に細かいことを調べようとしたとき、

なかなか大変だったりしませんか？

-->

---

# 気軽に使えるけど、本当に理解して使うのは難しい

<!--

「気軽に使えるけど、本当に理解して使うのは難しい」

少なくとも私は、しっかり理解して使えているとは言えない状況でした。

ドキュメントに書いてある通りにざっと使う分には、まあ困ることもあまりありません。

が、そこからちょっと外れたとき。

機能によってはコードちょっと追うだけでわかることもありますが、

コードを理解するのがそもそも難しい場合もありました。

気軽に使えるようにするために、Eloquentの中ではいろいろと複雑なことをしているんです。

-->

---

```php
<?php

namespace App\Models;

// これはEloquent\Modelを継承したクラス
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    // ...
}
```

<!--

実際に私がどういうところで引っかかったか、いくつか事例を紹介します。

これはバージョン<VERSION>での、デフォルトのUserモデルです。

細かいところ端折ってますが。

さて、$fillableというプロパティがあります。

ご存知の通り、ここで指定されているプロパティ以外には一括割り当てができない、

というものです。

TODO: この辺からスライド分けたり？

では具体的にこれは、どこまで有効なのでしょうか？

テスト環境のアカウントのパスワード忘れちゃって、tinkerからupdateでパスワードを変
更しようとしてエラーになった、なんてことは割とみなさんご経験あるんじゃないでしょ
うか。

パスワードは、$fillableに設定されていませんから。

ではcreateでも同様にエラーになるのか。プロパティへの代入で設定してsaveするパター
ンでは？

そもそもEloquentでINSERTする方法はそれだけでしょうか？　すべての方法を知っていま
すか？

-->

---

```php
<?php

# TODO: datesとかがいい？
```

<!--

アクセサはまあわかる、プロパティにアクセスしたときに多分処理してる。

では、キャストはどうなのか？　アクセサと同様？　違うタイミング？

ミューテタは？

-->

---

<!--

リレーションにクエリとしてアクセスした場合って中ではなにが起こってるの？

リレーション経由でupdateOrCreateしたときに、

？？？

-->

---

```php
<?php

use App\Models\User;

User::insert([
  ['name' => 'tarou', 'email' => 'tarou@example.com', 'password' => bcrypt('...')],
  ['name' => 'jirou', 'email' => 'jirou@example.com', 'password' => bcrypt('...')],
]);
```

<!--

あるいは、複数行を一気に挿入したくて、insertを使ったら、

-->

---

```tinker
>>> User::all()
=> Illuminate\Database\Eloquent\Collection {#3880
     all: [
       App\Models\User {#4502
         id: 1,
         name: "tarou",
         email: "tarou@example.com",
         email_verified_at: null,
         #password: "$2y$10$cwEulK.T4gTcHuycWBIpXOATx7gQFZqzpxEXqhc2wUb7MqSp51jpW",
         #remember_token: null,
         created_at: null,
         updated_at: null,
       },
       App\Models\User {#4240
         id: 2,
         name: "jirou",
         email: "jirou@example.com",
         email_verified_at: null,
         #password: "$2y$10$AlitastY/2.g.sTzw8EnGuBfrCxoq21Zvc7exttPmn8dE6VEKhRQm",
         #remember_token: null,
         created_at: null,
         updated_at: null,
       },
     ],
   }
```

<!--

ご覧の通り、created_at, updated_atが保存されていません。

なぜでしょう？

-->

---

<!--

必要になったときに、該当のメソッドと、関連するいくつかのメソッドを調べる、でもも
ちろん理解は深まっていく。

けど、一度全貌を理解してみたら、きっとこういう細かい疑問の答えも、すぐ出てくるよ
うになるんじゃないかな。

-->

---

# ということで(このセッションで)

* 巨大で複雑なEloquentの全貌をどうにか理解して、
* Eloquentで複雑なことをしたり細かいことを調べるときに、
* 大変な思いをせず、できるだけ簡単にできるようになる。

<!--

XXX じゃなくて、このセッションではEloquentをいろいろ見て理解していこう、みたいな
方がいい？

-->

---

<!--

実際に調査を始める前にもう一つ。こうやって調査して、全貌がわかると、いいことがあるのでしょうか？

これはもういいのか。

-->

---

# そもそもEloquentって本当に「巨大」なの？

* コードの規模の計測は難しい
* とはいえ、大きいか小さいかくらいは、ある程度の共通認識ができそう

<!--
-->

---

# 数えてみよう

Eloquent\Modelを継承したクラス(以下「モデル」)のメソッド数を数えてみます。

```php
count((new ReflectionClass(new class extends Illuminate\Database\Eloquent\Model {}))->getMethods())
```

これをtinkerで実行してみると、

<!--

TODO: Eloquent\Modelを継承したクラスのメソッド数を数えてみましょう～、みたいな。

-->

---

# 350

<!--

350という数値が出ました。protectedメソッドも含むんですが。

とはいえ、まあ十分多いのではないでしょうか。

仕事で350メソッドもあるクラスを書いたら、多分レビューは通らないと思います。

さて、実際350ってどのくらいでしょう？

-->

---

<!--

(メソッド一覧)

-->

---

<!--

以上です。もうわけがわかりません。

本当に恐しいのは、すでに気付いている人もいると思うんですが、実際にはこれ以上のメソッドがあることですね。


-->

---

<!--

とにかく、350あるいはそれ以上のメソッドを、端から端までぜんぶ読むのはちょっと大変そうです。

-->

---

# ではどうするか？

<!--
-->

