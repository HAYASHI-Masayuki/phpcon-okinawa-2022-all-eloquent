---
title: 30分で理解するEloquentの巨大な全貌
author: HAYASHI Masayuki
---

# 30分で理解するEloquentの巨大な全貌

HAYASHI Masayuki

<!--

こんにちは。林と申します。
さて、「30分で理解するEloquentの巨大な全貌」というタイトルで、
今日はお話させていただきます。
よろしくお願いします。

-->

<style scoped>
h1, p {
  text-align: center;
}
</style>

---

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
ドキュメントに書いてある通りにざっと使う分には、
まあ困ることもあまりありません。
が、そこからちょっと外れたとき。
機能によってはコードちょっと追うだけでわかることもありますが、
コードを理解するのがそもそも難しい場合もありました。
気軽に使えるようにするために、
Eloquentの中ではいろいろと複雑なことをしているんです。

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
これはデフォルトのUserモデルです。細かいところ端折ってますが。
さて、$fillableというプロパティがあります。ご存知の通り、
ここで指定されているプロパティ以外には一括代入ができない、
というものです。

-->

---

```php
<?php

// createの場合は？
User::create([
    'name'              => 'tarou',
    'email'             => 'tarou@example.com',
    'password'          => bcrypt('...'),
    'email_verified_at' => now()
]);

// setterの場合は？
$user = new User();
$user->name              = 'tarou';
$user->email             = 'tarou@example.com';
$user->password          = bcrypt('...');
$user->email_verified_at = now();
$user->save();
```

<!--

具体的にこれは、どこまで有効なのでしょうか？
公式ドキュメントには、一括代入の脆弱性から保護すると書いてあります。
実際にはどのように保護されているのでしょうか。
createの例は該当しそうです。
プロパティに代入しているのであれば、きっと大丈夫でしょう。

しかしEloquentでINSERTする方法はそれだけでしょうか？

-->

---

```php
<?php

$user = User::firstOrCreate([
    'email'             => 'tarou@example.com',
    'email_verified_at' => null,
], [
    'name'              => 'tarou',
    'password'          => bcrypt('...'),
    'email_verified_at' => now(),
]);
```

<!--

さて、firstOrCreateの第1引数で使う場合は？　第2引数で使う場合は？
email_verified_atをこういう風に使うことはないと思いますが、
このようなパターン自体はありうるのではないでしょうか。

-->

---

```php
<?php

$user->posts()->updateOrCreate(['user_id' => $user->id], $attributes);
```

<!--

私は昔、このようなコードを書いてしまったことがあります。
このコードのどこがおかしいか、すぐにわかりますか？
詳しい人であれば、私がなぜ間違ったのか、
その理由さえ一目見ただけでわかるかもしれません。

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

こういう感じで引っかかったとき、大抵の場合は該当のメソッドと、
関連するいくつかのメソッドを調べるだけでも答えを得ることはできます。
でもそうでない場合もあります。
そういうときでも、しっかりEloquentの全貌を理解していれば、
よりスムーズに答えを得られるのではないかと思います。

-->

---

# このセッションでは

* 巨大で複雑なEloquentの全貌をどうにか理解して、
* Eloquentで複雑なことをしたり細かいことを調べるときに、
* 大変な思いをせず、できるだけ簡単にできるようになる。

<!--

(スライド読む・切り替え)
という感じでやっていきたいと思います。

-->

---

# そもそもEloquentって本当に「巨大」なの？

- コードの規模の計測は難しい
- とはいえ、大きいか小さいかくらいは、ある程度の共通認識ができそう

<!--

(スライド読む)
さて、タイトルから始まってずっと、Eloquentは巨大だと話してきました。
ですが実際どうでしょうか？
よく言われる通り、コードの規模の計測は難しいです。
とはいえ規模が大きいか小さいか、くらいであれば、
ある程度共通認識を作ることもできそうです。

-->

---

# Eloquentの「範囲」

* \Illuminate\Database\Eloquent\Modelを継承したクラス→Eloquentのモデル

<!--

規模について考える前にもう一つだけ。
このセッションで話す、
「Eloquent」の範囲について、前提となる認識を作りたいと思います。
(切り替え)
といっても簡単です。このセッションでは、
\Illuminate\Database\Eloquent\Modelを継承したクラスについて、
Eloquentのモデル、として扱い、それをEloquentの範囲とします。

-->

---

# 「範囲外」

* \Illuminate\Database以下でもEloquent以外は「範囲外」
* make:modelコマンド
* ファクトリ

<!--

つまり、
(切り替え)
\Illuminate\Database以下でもEloquent以外の名前空間のものや、
(切り替え)
artisanのmake:modelコマンド、
(切り替え)
あるいはファクトリ。

これらは、Eloquentと深い関係にありますが、範囲外とします。

-->

---

# 数えてみよう

```php
count((new ReflectionClass(new class extends Illuminate\Database\Eloquent\Model {}))->getMethods())
```

これをtinkerで実行してみると、

<!--

さて、それではまず、Eloquentのモデルのメソッド数を数えてみます。
コードの規模を考える上で、いちばん手軽なのはコードの行数ですが、
特に比較対象がない場合に行数だけから規模を考えるのは、
なかなか難しい気がするので、もうちょっとマシそうな、メソッド数です。

tinkerで、リフレクションを使って、
特にメソッドを追加していないモデルクラスのメソッド数を数えてみます。
手もとにLaravelを動かせる環境がある人は、
ぜひ実際に試してみてください。

-->

---

# 350

<!--

350という数値が出ました。protectedメソッドも含むんですが。
バージョンによって違ってはくると思いますが、
とはいえ、まあ十分多いのではないでしょうか。
仕事で350メソッドもあるクラスを書いたら、
多分レビューは通らないと思います。
さて、実際350ってどのくらいでしょう？

-->

<style scoped>
h1, p {
  text-align: center;
}
</style>

---

`__call` `__callStatic` `__construct` `__get` `__isset` `__set` `__sleep` `__toString` `__unset` `__wakeup` `addCastAttributesToArray` `addDateAttributesToArray` `addGlobalScope` `addMutatedAttributesToArray` `addObservableEvents` `all` `append` `asDate` `asDateTime` `asDecimal` `asJson` `asTimestamp` `attributesToArray` `belongsTo` `belongsToMany` `boot` `bootIfNotBooted` `bootTraits` `booted` `booting` `broadcastChannel` `broadcastChannelRoute` `cacheMutatedAttributes` `callNamedScope` `castAttribute` `castAttributeAsEncryptedString` `castAttributeAsJson` `clearBootedModels` `created` `creating` `decrement` `decrementQuietly` `delete` `deleteOrFail` `deleteQuietly` `deleted` `deleting` `destroy` `deviateClassCastableAttribute` `encryptUsing` `escapeWhenCastingToString` `fill` `fillJsonAttribute` `fillable` `fillableFromArray` `filterModelEventResults` `finishSave` `fireCustomModelEvent` `fireModelEvent` `flushEventListeners` `forceDelete` `forceFill` `forwardCallTo` `forwardDecoratedCallTo` `fresh` `freshTimestamp`

<!--

(メソッド一覧)

-->

---

`freshTimestampString` `fromDateTime` `fromEncryptedString` `fromFloat` `fromJson` `getActualClassNameForMorph` `getArrayAttributeByKey` `getArrayAttributeWithValue` `getArrayableAppends` `getArrayableAttributes` `getArrayableItems` `getArrayableRelations` `getAttribute` `getAttributeFromArray` `getAttributeMarkedMutatorMethods` `getAttributeValue` `getAttributes` `getAttributesForInsert` `getCastType` `getCasts` `getChanges` `getClassCastableAttributeValue` `getConnection` `getConnectionName` `getConnectionResolver` `getCreatedAtColumn` `getDateFormat` `getDates` `getDirty` `getEnumCastableAttributeValue` `getEventDispatcher` `getFillable` `getForeignKey` `getGlobalScope` `getGlobalScopes` `getGuarded` `getHidden` `getIncrementing` `getKey` `getKeyForSaveQuery` `getKeyForSelectQuery` `getKeyName` `getKeyType` `getMorphClass` `getMorphs` `getMutatedAttributes` `getMutatorMethods` `getObservableEvents` `getOriginal` `getOriginalWithoutRewindingModel` `getPerPage` `getQualifiedCreatedAtColumn` `getQualifiedKeyName`

<!--

(メソッド一覧)

-->

---

`getQualifiedUpdatedAtColumn` `getQueueableConnection` `getQueueableId` `getQueueableRelations` `getRawOriginal` `getRelation` `getRelationValue` `getRelations` `getRelationshipFromMethod` `getRouteKey` `getRouteKeyName` `getTable` `getTouchedRelations` `getUpdatedAtColumn` `getVisible` `guard` `guessBelongsToManyRelation` `guessBelongsToRelation` `handleLazyLoadingViolation` `handleLazyLoadingViolationUsing` `hasAppended` `hasAttributeGetMutator` `hasAttributeMutator` `hasAttributeSetMutator` `hasCast` `hasChanges` `hasGetMutator` `hasGlobalScope` `hasMany` `hasManyThrough` `hasNamedScope` `hasOne` `hasOneThrough` `hasSetMutator` `increment` `incrementOrDecrement` `incrementQuietly` `initializeTraits` `insertAndSetId` `is` `isClassCastable` `isClassDeviable` `isClassSerializable` `isClean` `isCustomDateTimeCast` `isDateAttribute` `isDateCastable` `isDateCastableWithCustomFormat` `isDecimalCast` `isDirty` `isEncryptedCastable` `isEnumCastable` `isFillable` `isGuardableColumn` `isGuarded` `isIgnoringTouch` `isImmutableCustomDateTimeCast`

<!--

(メソッド一覧)

-->

---

`isJsonCastable` `isNot` `isRelation` `isStandardDateFormat` `isUnguarded` `joiningTable` `joiningTableSegment` `jsonSerialize` `load` `loadAggregate` `loadAvg` `loadCount` `loadExists` `loadMax` `loadMin` `loadMissing` `loadMorph` `loadMorphAggregate` `loadMorphAvg` `loadMorphCount` `loadMorphMax` `loadMorphMin` `loadMorphSum` `loadSum` `makeHidden` `makeHiddenIf` `makeVisible` `makeVisibleIf` `mergeAttributesFromAttributeCasts` `mergeAttributesFromCachedCasts` `mergeAttributesFromClassCasts` `mergeCasts` `mergeFillable` `mergeGuarded` `morphEagerTo` `morphInstanceTo` `morphMany` `morphOne` `morphTo` `morphToMany` `morphedByMany` `mutateAttribute` `mutateAttributeForArray` `mutateAttributeMarkedAttribute` `newBaseQueryBuilder` `newBelongsTo` `newBelongsToMany` `newCollection` `newEloquentBuilder` `newFromBuilder` `newHasMany` `newHasManyThrough` `newHasOne` `newHasOneThrough` `newInstance` `newModelQuery` `newMorphMany` `newMorphOne` `newMorphTo` `newMorphToMany` `newPivot` `newQuery` `newQueryForRestoration`

<!--

(メソッド一覧)

-->

---

`newQueryWithoutRelationships` `newQueryWithoutScope` `newQueryWithoutScopes` `newRelatedInstance` `newRelatedThroughInstance` `normalizeCastClassResponse` `observe` `offsetExists` `offsetGet` `offsetSet` `offsetUnset` `on` `onWriteConnection` `only` `originalIsEquivalent` `parseCasterClass` `performDeleteOnModel` `performInsert` `performUpdate` `preventLazyLoading` `preventsLazyLoading` `push` `qualifyColumn` `qualifyColumns` `query` `refresh` `registerGlobalScopes` `registerModelEvent` `registerObserver` `reguard` `relationLoaded` `relationsToArray` `removeObservableEvents` `replicate` `replicateQuietly` `replicating` `resolveCasterClass` `resolveChildRouteBinding` `resolveChildRouteBindingQuery` `resolveConnection` `resolveRelationUsing` `resolveRouteBinding` `resolveRouteBindingQuery` `resolveSoftDeletableChildRouteBinding` `resolveSoftDeletableRouteBinding` `retrieved` `save` `saveOrFail` `saveQuietly` `saved` `saving` `serializeClassCastableAttribute` `serializeDate` `setAppends` `setAttribute` `setAttributeMarkedMutatedAttributeValue`

<!--

(メソッド一覧)

-->

---

`setClassCastableAttribute` `setConnection` `setConnectionResolver` `setCreatedAt` `setDateFormat` `setEnumCastableAttribute` `setEventDispatcher` `setHidden` `setIncrementing` `setKeyName` `setKeyType` `setKeysForSaveQuery` `setKeysForSelectQuery` `setMutatedAttributeValue` `setObservableEvents` `setPerPage` `setRawAttributes` `setRelation` `setRelations` `setTable` `setTouchedRelations` `setUpdatedAt` `setVisible` `syncChanges` `syncOriginal` `syncOriginalAttribute` `syncOriginalAttributes` `throwBadMethodCallException` `toArray` `toJson` `totallyGuarded` `touch` `touchOwners` `touches` `transformModelValue` `unguard` `unguarded` `unsetConnectionResolver` `unsetEventDispatcher` `unsetRelation` `unsetRelations` `update` `updateOrFail` `updateQuietly` `updateTimestamps` `updated` `updating` `usesTimestamps` `wasChanged` `with` `withoutBroadcasting` `withoutEvents` `withoutRelations` `withoutTouching` `withoutTouchingOn`

<!--

(メソッド一覧)

以上です。見るだけでつらい感じになりますね。
これでEloquentが巨大であるということには、
ひとまず同意いただけるのではないでしょうか。
なおすでに気付いている人もいると思うんですが、
実際にはEloquentには、これ以上のメソッドがあります。

しかし、350あるいはそれ以上のメソッドを、
端から端までぜんぶ読むのはちょっと大変そうです。
大変以前に、今日のセッションの時間は30分しかないので、
まあ不可能ですね。
ではどうやって、30分でEloquentの全貌を理解しましょう？

-->

---

# ではどうするか？

- 分割して理解する
- 抽象化して把握する

<!--

我々はプログラマなので、
大きなもの、複雑なものにどう対処すればいいかは知っています。
分割して少しずつ理解したり、
抽象化して大きなものも把握しやすくしたり、
そういう手法をあれこれ使って、なんとかしてきたいと思います。

-->

