---
title: 30分で理解するEloquentの巨大な全貌
author: HAYASHI Masayuki
---

# 30分で理解するEloquentの巨大な全貌

HAYASHI Masayuki

<style scoped>
h1, p {
  text-align: center;
}
</style>

<!--

こんにちは。林と申します。
さて、「30分で理解するEloquentの巨大な全貌」というタイトルで、
今日はお話させていただきます。
よろしくお願いします。

-->

---

# Eloquentしっかり理解して使えてますか？

<!--

(1章)

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

TODO: ついでにこの辺で、クラスの呼び方について話しておく？　I\Dは省略とか。

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

<style scoped>
h1, p {
  text-align: center;
}
</style>

<!--

350という数値が出ました。protectedメソッドも含むんですが。
バージョンによって違ってはくると思いますが、
とはいえ、まあ十分多いのではないでしょうか。
仕事で350メソッドもあるクラスを書いたら、
多分レビューは通らないと思います。
さて、実際350ってどのくらいでしょう？

-->

---

`__call` `__callStatic` `__construct` `__get` `__isset` `__set` `__sleep` `__toString` `__unset` `__wakeup` `addCastAttributesToArray` `addDateAttributesToArray` `addGlobalScope` `addMutatedAttributesToArray` `addObservableEvents` `all` `append` `asDate` `asDateTime` `asDecimal` `asJson` `asTimestamp` `attributesToArray` `belongsTo` `belongsToMany` `boot` `bootIfNotBooted` `bootTraits` `booted` `booting` `broadcastChannel` `broadcastChannelRoute` `cacheMutatedAttributes` `callNamedScope` `castAttribute` `castAttributeAsEncryptedString` `castAttributeAsJson` `clearBootedModels` `created` `creating` `decrement` `decrementQuietly` `delete` `deleteOrFail` `deleteQuietly` `deleted` `deleting` `destroy` `deviateClassCastableAttribute` `encryptUsing` `escapeWhenCastingToString` `fill` `fillJsonAttribute` `fillable` `fillableFromArray` `filterModelEventResults` `finishSave` `fireCustomModelEvent` `fireModelEvent` `flushEventListeners` `forceDelete` `forceFill` `forwardCallTo` `forwardDecoratedCallTo` `fresh` `freshTimestamp`

<!--

(メソッド一覧)

(2回目)
こちらアルファベット順に並んでいるのですが……。
fill, fillable, そしてfire..., でも次はflushですね、firstはありません。
(+1ページ)

-->

---

`freshTimestampString` `fromDateTime` `fromEncryptedString` `fromFloat` `fromJson` `getActualClassNameForMorph` `getArrayAttributeByKey` `getArrayAttributeWithValue` `getArrayableAppends` `getArrayableAttributes` `getArrayableItems` `getArrayableRelations` `getAttribute` `getAttributeFromArray` `getAttributeMarkedMutatorMethods` `getAttributeValue` `getAttributes` `getAttributesForInsert` `getCastType` `getCasts` `getChanges` `getClassCastableAttributeValue` `getConnection` `getConnectionName` `getConnectionResolver` `getCreatedAtColumn` `getDateFormat` `getDates` `getDirty` `getEnumCastableAttributeValue` `getEventDispatcher` `getFillable` `getForeignKey` `getGlobalScope` `getGlobalScopes` `getGuarded` `getHidden` `getIncrementing` `getKey` `getKeyForSaveQuery` `getKeyForSelectQuery` `getKeyName` `getKeyType` `getMorphClass` `getMorphs` `getMutatedAttributes` `getMutatorMethods` `getObservableEvents` `getOriginal` `getOriginalWithoutRewindingModel` `getPerPage` `getQualifiedCreatedAtColumn` `getQualifiedKeyName`

<!--

(メソッド一覧)

(2回目)
fの後、gの最初が、getActual...です、単体のgetはありません。
(+4ページ)

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

(2回目)
wasChangedの後はwithです、whereありませんね。
(+4ページ)

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

---

# TODO

<!--

(2章)

それでは早速調べていきたいと思います。
しかし一体どう進めるのがよいでしょう？
こういうときにあまり細かいところから見ていっても、
なかなか全体像は見えてこない気がします。
ので、大きな個所から攻めたいところです。

-->

---

# Eloquentはどのように使うか

```php
<?php

$users = User::where('email', 'like', '%@example.com')->get();
$user  = User::where('id', $id)->first();

echo $user->name;
$user->update(['password' => bcrypt('...')]);
```

<!--

どういう使い方をするか、から考えてみます。
Eloquentの使い方としては大きく分けて、
2つあるんじゃないかなと思います。
1つは、whereなどで検索して、getやfirstで行を取得する。
もう1つは、取得した行を使っていく。各カラムの内容を表示したり、
updateやdeleteで行を操作したり、ですね。
今回は前者から見てみます。

TODO: 前者から見てみます～、のところもうちょっとしっかり言う。

さて、先程Eloquent\Modelを継承したクラスに実装されている
メソッドを見てみました。350もありました。
が、実はあの中には、whereもgetも、firstもありません。
見直してみましょう。

(メソッド一覧に戻る: -8ページ)

-->

---

# Eloquent\Modelにはwhere, get, firstはない

<!--

Eloquent\Modelにはwhere, get, firstはありませんでした。

ご存知の方はご存知だと思うのですが、
この辺のメソッドはEloquent\Modelには直接実装されていません。
ではどうなっているのか？

-->

---

User::where
→ User::__callStatic
→ User->__call
→ Eloquent\Builder::where

<!--

あるクラスの、実装されていない静的メソッドを呼び出そうとしたとき、
マジックメソッド__callStaticが実装されていれば、
そちらが呼び出されます。

今回Eloquent\Modelにwhereメソッドはないので、
__callStaticが呼び出されるわけです。

そこから紆余曲折あって、最終的にはEloquent\Builderというクラスの
インスタンスが生成され、そのメソッドが呼び出されます。

Eloquent\Modelは、このような形で一部の操作をEloquent\Builderに
委譲しています。

Eloquent\Builderのメソッドを見てみましょう。
今度はたったの162メソッドです。

TODO: ここ、結論から！

-->

---

`__call` `__callStatic` `__clone` `__construct` `__get` `addHasWhere` `addNestedWiths` `addNewWheresWithinGroup` `addTimestampsToUpsertValues` `addUpdatedAtColumn` `addUpdatedAtToUpsertColumns` `addWhereCountQuery` `applyScopes` `baseSole` `callNamedScope` `callScope` `canUseExistsForExistenceCheck` `chunk` `chunkById` `chunkMap` `clone` `combineConstraints` `create` `createNestedWhere` `createSelectWithConstraint` `cursor` `cursorPaginate` `cursorPaginator` `decrement` `defaultKeyName` `delete` `doesntHave` `doesntHaveMorph` `each` `eachById` `eagerLoadRelation` `eagerLoadRelations` `enforceOrderBy` `ensureOrderForCursorPagination` `find` `findMany` `findOr` `findOrFail` `findOrNew` `first` `firstOr` `firstOrCreate` `firstOrFail` `firstOrNew` `firstWhere` `forceCreate` `forceDelete` `forwardCallTo` `forwardDecoratedCallTo` `fromQuery` `get` `getBelongsToRelation` `getEagerLoads` `getGlobalMacro` `getMacro` `getModel` `getModels` `getOriginalColumnNameForCursorPagination` `getQuery` `getRelation` `getRelationWithoutConstraints` `groupWhereSliceForScope` `has` `hasGlobalMacro` `hasMacro` `hasMorph` `hasNamedScope` `hasNested` `hydrate` `increment` `isNestedUnder` `latest` `lazy` `lazyById` `lazyByIdDesc` `make` `mergeConstraintsFrom`

<style scoped>
code:nth-child(45),
code:nth-child(56) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

Eloquent\Builderに実装されているメソッドです。

firstがあります。
getもありますね。

-->

---

`newModelInstance` `oldest` `onDelete` `orDoesntHave` `orDoesntHaveMorph` `orHas` `orHasMorph` `orWhere` `orWhereBelongsTo` `orWhereDoesntHave` `orWhereDoesntHaveMorph` `orWhereHas` `orWhereHasMorph` `orWhereMorphRelation` `orWhereMorphedTo` `orWhereNot` `orWhereNotMorphedTo` `orWhereRelation` `orderedLazyById` `paginate` `paginateUsingCursor` `paginator` `parseNameAndAttributeSelectionConstraint` `parseWithRelations` `pluck` `prepareNestedWithRelationships` `qualifyColumn` `qualifyColumns` `registerMixin` `relationsNestedUnder` `removedScopes` `requalifyWhereTables` `scopes` `setEagerLoads` `setModel` `setQuery` `simplePaginate` `simplePaginator` `sole` `soleValue` `tap` `throwBadMethodCallException` `toBase` `unless` `update` `updateOrCreate` `upsert` `value` `valueOrFail` `when` `where` `whereBelongsTo` `whereDoesntHave` `whereDoesntHaveMorph` `whereHas` `whereHasMorph` `whereKey` `whereKeyNot` `whereMorphRelation` `whereMorphedTo` `whereNot` `whereNotMorphedTo` `whereRelation` `with` `withAggregate` `withAvg` `withCasts` `withCount` `withExists` `withGlobalScope` `withMax` `withMin` `withOnly` `withSum` `withWhereHas` `without` `withoutEagerLoad` `withoutEagerLoads` `withoutGlobalScope` `withoutGlobalScopes`

<style scoped>
code:nth-child(51) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

whereもありました。

-->

---

- Eloquent\Model(を継承したクラス) ... 350メソッド
- Eloquent\Builder ... 162メソッド
- 計 ... 512メソッド

<!--

Eloquent\Modelと合わせて、メソッドが500を超えています。
とはいえこれで一通り、よく使うメソッドは揃ったかな、
と思いきや、実はまだ足りないものがあります。

お気付きになられたでしょうか？
selectも、joinも、groupByもorderByも、今の一覧にはありませんでした。

どうなっているのでしょう？　まあ、予想つきますよね。

-->

---

Eloquent\Builder->select
→ Eloquent\Builder->__call
→ Illuminate\Database\Query\Builder::select

<!--

Eloquent\Builderからは、さらに別のクラスのメソッドが
呼び出されるようになっています。
Illuminate\Database\Query\Builderというクラスです。
Eloquent\ModelからEloquent\Builderへの委譲同様、
マジックメソッドによって、Eloquent\Builderにないメソッドは
Query\Builderのものが呼び出されるようになっています。
そして今度こそ、私たちが普段使っている機能が一通り揃っています。

さすがにしつこいのでメソッドの一覧はもう出しませんが、
Query\Builderには200を超えるメソッドがあり、
select, join, groupBy, orderByほか、普段Eloquentで
SQLを操作する際に使うメソッドは、ほとんどここに実装されています。

-->

---

# User::whereが返すのはEloquent\Builder

```php
<?php

echo get_class(User::where('id', $id));
// → 'Illuminate\Database\Eloquent\Builder'
```

<!--

IDE Helperなどを入れていい感じに補完が効くようにしている方は
よくご存知かと思いますが、
Eloquentのクラスから静的にメソッドを実行して、
戻ってくるのはEloquent\Builderです。
そこからメソッドをチェーンしていくときに使えるのは、
Eloquent\BuilderかQuery\Builderのメソッドというわけです。

-->

---

# Query\Builderを知る

- 200を超えるメソッドの半数以上が、自身を返すメソッド
  - 大半の機能がメソッドチェーンでSQLを組み立てるもの
- 残りの多くは、取得した行、そのコレクションを返す

<!--

Query\Builderをちょっと見てみます。
Query\Builderには200を超えるメソッドがありますが、
そのうち半数以上が自身を返すメソッドです。

残りは結構ばらばらですが、取得した行や、
そのコレクションを返すものがかなり多い感じです。

この辺を見るだけでも、Query\Builderの姿、
SQLクエリをメソッドチェーンで表現する、そのクエリを実行する。
そういう仕事のためのクラスになっているのがわかるかと思います。

-->

---

# Query\BuilderはEloquentの一部……ではない

- 単純に、名前空間が違う
  - Eloquentは、Illuminate\Database\Eloquent以下
  - Query\Builderは、Illuminate\Database\Query\Builder

<!--

さて、Query\Builderも興味深いクラスではあるのですが、
実はこれ、Eloquentの範囲外だと思うんですよね。
単純に、Illuminate\Database\Eloquent以下のクラスではないので……。

ですのでこれ以上深堀はせず、進めます。

-->

---

# なぜ直接Query\Builderを使わないのか？

* Eloquent\Builderがなにを実装しているかを見ればよさそう
  * Query\Builderのメソッドをオーバーライドしているもの
  * それ以外＝Eloquent\Builder独自のもの

<!--

Eloquent\Modelが、SQLの処理をEloquent\Builder経由でQuery\Builderに
任せている。ここまでは問題ないですよね。
しかしそうなったときに一つ気になることがあります。
なぜ、Eloquent\Builder経由なんでしょうか？
Eloquent\Modelが直接Query\Builderを使わないのはなぜでしょうか？

(めくる)

それを知るには、
Eloquent\Builderがなにを実装しているかを見ればよさそうです。

(めくる*2)

Eloquent\Builderのメソッドには、Query\Builderをオーバーライド
しているものと、そうでないもの、Eloquent\Builder独自のものがあります。

前者から見てみましょう。Query\Builderにあるのにわざわざオーバーライド
していることには、意味がありそうです。

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`

<!--

まあいろいろあるんですが、目立つのは、whereやfind, get,
あとはlatestあたりですかね。
これらのメソッドが、なぜQuery\Builderのものではなく、独自のものを
使うようになっているのか、がわかれば、
Eloquent\Builderの存在意義についてもわかりそうです。

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`

<style scoped>
li li:nth-child(2) code:nth-child(1),
li li:nth-child(3) code {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

TODO: もうちょっと整理
TODO: 主キーの知識とかのところ、もうちょっと強調！

findやlatestは、これは主キーやタイムスタンプ、created_at, updated_at
ですね、この辺が関係してくる機能です。
つまりこれは、Eloquentが単なるクエリビルダと違って、テーブルの情報、
この場合は主キーがなんであるか、タイムスタンプのカラムがなんであるか、
みたいなことを知っているために、Query\Builderのそれが実装しているように、
とりあえずidなりcreated_atなりをデフォルトとして既定する、ではなく、
知っている情報に従って、適切なカラムを使うようにできる、
それが理由です。

Eloquent\Builderのfindは、Eloquent\Modelの$primaryKeyに設定された
主キーを使います。latestは、定数CREATED_ATに設定された作成時に
自動設定されるカラムを使います。

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`

<style scoped>
li li:nth-child(1) code:nth-child(1),
li li:nth-child(2) code:nth-child(2) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

whereやgetは、またちょっと別の理由で、独自に実装されています。
これはQuery\Builderにはない、Eloquent独自の機能、
グローバルスコープやリレーションのため。

また、戻り値をstdClassではなくモデルのクラスで取得したり、
さらにコレクションも、ただのコレクションではなくEloquentの独自の
コレクションを使うようになっています。

-->

---

# Eloquent\Builderのメソッド(2)

- Eloquent\Builderだけにあるもの
  - 名前に`key`とか`timestamp`を含むもの
  - 名前に`scope`を含むもの
  - 名前に`eagerLoad` `relation` `with`を含むもの
<!-- 
/*
- 共通で使用しているBuildQueriesトレイトにあるもの
  - chunk, each, lazy
  - first
  - paginator, simplePaginator
  */
-->

<!--

Eloquent\Builderにだけある、新規のメソッドについて見てみても、
同じような感じです。
やはり、Eloquentがテーブルのカラムの情報を知っていることに関するもの、
グローバルスコープ、リレーションといったEloquent独自の機能、
それらに関連するメソッドがほとんどです。

つまり単純に、Eloquent\Builderの存在意義は、Query\Builderにはない、
Eloquent独自の機能のうち、Query\Builderに関係の強いものを
まとめておく、といった感じと考えてよさそうです。

TODO: createとか、BuildQueriesとか、その辺の話は？　時間足りる？

-->

---

# ここまでにわかったこと

* Eloquentの機能のうち、少なくない部分がEloquent\Builder, Query\Builderにある
* Builder 2つを合わせたメソッド数は、Eloquent\Modelのメソッド数に匹敵
* →Eloquentの半分は、クエリビルダでできている？

<!--

ここまででわかったことを簡単にまとめます。

(めくる)

Eloquentの機能のうち、少なくない部分がEloquent\Builder,
Query\Builderによるものでした。
SQLを組み立てて、実行して、というあたりはほぼそうです。

(めくる)

規模的にも、Eloquent\Modelが350メソッドあったのに対し、
Builder 2つを合わせると同等以上ありました。
実は2つのBuilderから同じトレイトを使っていたりして、
単純計算で数えるのもちょっと違うんですが。

(めくる)

ということで、Eloquentの半分近くはクエリビルダでできている、
と考えてよいのではないでしょうか。
そう考えると、Eloquentが意外とシンプルに見えてきます。

-->

---

```php
<?php

User::where('id', $id)      // これはEloquent\Builder
    ->select('id', 'email') // これはQuery\Builderにしかないはず？
    ->get();                // これはEloquent\Builderのはず
```

<!--

TODO: 時間足りなければ削除。逆に余裕あれば、もうちょっとわかりやすくしても？

ちなみにEloquent\Builderをチェーンしつつ
Query\Builderのメソッドを使えるの、
最初どういう実装になっているのかわからなかったんですが、

-->

---

```php
<?php

namespace Illuminate\Database\Eloquent;

class Builder
{
    public function __call($method, $parameters)
    {
        // ...

        // $this->queryはQuery\Builder
        $this->forwardCallTo($this->query, $method, $parameters);

        // 返すのは、Eloquent\Builder
        return $this;
    }
}
```

<!--

こんな感じでした。

単純に、委譲した結果をそのまま返すのではなく、
返すのはEloquent\Builder自身になっているんです。

面白い実装ですが、こうなると気になってくるのが、
自身を返す、チェーンするメソッド以外の場合どうなるのか、です。

-->

---

```php
<?php

namespace Illuminate\Database\Eloquent;

class Builder
{
    protected $passthru = [
        'aggregate', 'average', 'avg', 'count', 'dd', 'doesntExist', 'dump', 'exists',
        'explain', 'getBindings', 'getConnection', 'getGrammar', 'insert', 'insertGetId',
        'insertOrIgnore', 'insertUsing', 'max', 'min', 'raw', 'sum', 'toSql',
    ];

    public function __call($method, $parameters)
    {
        // ...

        if (in_array($method, $this->passthru)) {
            // 戻り値自体を返す！
            return $this->toBase()->{$method}(...$parameters);
        }

        $this->forwardCallTo($this->query, $method, $parameters);

        return $this;
    }
}
```

<!--

ちょっと上のコードに答えがありました。省略していた部分です。
一部の、$passthruで指定されたメソッド、これらに委譲する場合、
Eloquent\Builderを返すのではなく戻り値自体を返すようになっています。
よく工夫されています。その分、理解するのは大変ですが。

-->

---

# 

<!--



-->

---
