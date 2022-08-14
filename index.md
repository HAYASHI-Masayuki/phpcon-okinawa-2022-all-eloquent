---
title: 30分で理解するEloquentの巨大な全貌
author: HAYASHI Masayuki
---

<style>
.highlight {
  color: #f00;
  text-decoration: underline;
}
</style>

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

-->

---

# Eloquentの全貌を理解していれば、<br>問題解決もスムーズかも

<style scoped>
h1 {
  text-align: center;
}
</style>

<!--

今見てきたような疑問を解消したいとき、あるいは
Eloquentの仕様なのかバグなのかわからないような問題が起きたとき、
Eloquentの全貌を理解していれば、スムーズに解決できるのではないかと
思います。

-->

---

# どうやって理解するか

* ソースコード**ぜんぶ**読む
  - 無謀
* どういう機能があり、どう配置されているか、の把握に絞る
  - くらいならできそう

<!--

さて、ではどうやって理解していくか。

(めくる)
関連するコードをぜんぶ読んで理解、できればまあ
それがベストなのかもしれません。
とはいえこの巨大なコードベース、それは無謀です。

(めくる)
大事なのは全体像です。
なので、どういう機能があり、それらがどう配置されているか、
に絞って調べていきたいと思います。

-->

---

# Eloquentの「範囲」

* Illuminate\Database\Eloquent\Modelを継承したクラス→Eloquentのモデル

<!--

もう一つだけ。このセッションで話す、
「Eloquent」の範囲について、前提となる認識を作りたいと思います。

(めくる)
といっても簡単です。このセッションでは、
Illuminate\Database\Eloquent\Modelを継承したクラスについて、
Eloquentのモデル、として扱い、それをEloquentの範囲とします。

-->

---

# 「範囲外」

* Illuminate\Database以下でもEloquent以外は「範囲外」
* make:modelコマンド
* ファクトリ

<!--

つまり、

(めくる)
Illuminate\Database以下でもEloquent以外の名前空間のものや、

(めくる)
artisanのmake:modelコマンド、

(めくる)
あるいはファクトリ。

これらは、Eloquentと深い関係にありますが、範囲外とします。

-->

---

# 名前について1

- Illuminate\Database\Eloquent\Model -> Eloquent\Model
- Illuminate\Database\Query\Builder -> Query\Builder

<!--

なおクラス名の呼び方ですが、
基本的にIlluminate\Databaseまでは省略しようと思います。

-->

---

# 名前について2

- Eloquent\Modelを継承したオブジェクト -> モデル

<!--

また、Eloquent\Modelを継承したオブジェクトを「モデル」と呼びます。

-->

---

# そもそもEloquentって本当に「巨大」なの？

- コードの規模の計測は難しい
- とはいえ、大きいか小さいかくらいは、ある程度の共通認識ができそう

<!--

さて、タイトルから始まってずっと、Eloquentは巨大だと話してきました。
ですが実際どうでしょうか？
よく言われる通り、コードの規模の計測は難しいです。
とはいえ規模が大きいか小さいか、くらいであれば、
ある程度共通認識を作ることもできそうです。

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

さて、30分で本当に理解できるか、不安になってきました。

(2回目)
wasChangedの後はwithです、whereありませんね。
(+2ページ)

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

(2章)
さて、ではさっそく進めていきたいと思います。
まずは、Eloquentの使い方から考えていきます。

Eloquentの使い方としては大きく分けて、
2つあるんじゃないかなと思います。
1つは、whereなどで検索して、getやfirstで行を取得する。
もう1つは、取得した行を使っていく。各カラムの内容を表示したり、
updateやdeleteで行を操作したり、ですね。

まずは前者、whereで検索して、getで取得する、方から見てみます。

さて、先程Eloquent\Modelを継承したクラスに実装されている
メソッドを見てみました。350もありました。
が、実はあの中には、whereもgetも、firstもありません。
見直してみましょう。

(メソッド一覧に戻る: -6ページ)

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
getやfirstはEloquent\Builder自身に実装されていますが、
たとえばcount等はそうではありません。

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
forwardCallToの前のブロックですね。

$passthruで指定されたメソッド、これらに委譲する場合、
Eloquent\Builderを返すのではなく戻り値自体を返すようになっています。

よく工夫されています。その分、理解するのは大変ですが。

-->

---

# SQLクエリを組み立てて、実行する機能<br>→Eloquentの大きな部分

<!--

(3章)

SQLクエリを組み立てて、実行するというあたりがEloquentの大きな部分
と考えてよさそう、というところまでお話しました。

では次は、クエリを実行して取得したその結果、
それを使っていく方を見てみましょう。

-->

---

```tinker
>>> get_class(User::find(1))
=> "App\Models\User"

>>> get_parent_class(User::find(1))
=> "Illuminate\Foundation\Auth\User"

>>> get_parent_class(get_parent_class(User::find(1)))
=> "Illuminate\Database\Eloquent\Model"
```

<!--

ということで、取得したインスタンスの方を見ていきます。
これは当然、Eloquent\Modelのインスタンスです。
デフォルトのUserクラスは、ちょっと間に一つ挟んでるのですが。

-->

---

```php
<?php

$user = User::where('name', 'tarou')->first();
echo $user->name; // 'tarou'
$user->name = 'jirou';
echo $user->name; // 'jirou'
```

<!--

データベースのカラムはEloquentではアトリビュートとして扱います。
実装的には、
マジックメソッド__getで取得したり、同__setで設定したりします。

-->

---

# Eloquent\Model::__get, ::__setの中身

```php
<?php

namespace Illuminate\Database\Eloquent;

abstract class Model
{
    use Concerns\HasAttributes,

    public function __get($key)
    {
        return $this->getAttribute($key);
    }

    public function __set($key, $value)
    {
        $this->setAttribute($key, $value);
    }
}
```

<!--

中身はEloquent\Modelが使用しているHasAttributesトレイトの、
getAttribute, setAttributeです。

-->

---

# HasAttributesの調査に苦労した話

- 苦労した理由
  - 使い方ベースではなく、コードリーディングで調べようとした
* そもそもなぜHasAttributesを選んだか
  - 比較的大きい個所だから
  * まとまりのないEloquent\Modelよりはマシそうだったから

<!--

実はこの辺を調べたとき、最初とても苦労しました。というのも、
使い方ベースでなく、コードリーディング中心で調べてしまったんですね。

(めくる)
HasAttributesが大きな要素だということはわかっていました。
メソッド数は100を超えて、Eloquent\Modelから使われている
トレイトの中では最大です。Eloquent\Model本体でも130ほどなので。

(めくる)
Eloquent\Model本体には、分類しづらい雑多なメソッドが並んでいます。
実際簡単に分類できるものは、トレイトに移されて、
その残りが本体にある、という感じでしょう。

それと比べれば、HasAttributesはメソッドが100を超えるとはいっても、
ある程度整理されているはずなので、
もっと簡単に把握できるのではないかと思っていました。
が、これが難しかった。

-->

---

# HasAttributesはメソッドだけ見てもわからない

* 同じような動詞で始まるメソッドが多い
  - get, set, is, has, as, from
* 名詞もまた、同じようなものが多い
  - attribute, cast, mutator...
* 引数や戻り値を見ても、
  - 引数は、動詞ごとに似通っている
  - 戻り値はほぼプリミティブ型

<!--

なぜか。どの辺のメソッドが、HasAttributesの中心になるか、
がぱっとはわからなかったからです。

(めくる)
get, set, is, has, あとas, fromみたいな、
同じような動詞で始まるメソッドばかりだったり。

(めくる)
これらの対象となる名詞もまた、同じようなものがたくさん。
attribute, cast, mutator, ...

(めくる)
引数や戻り値で見てもなかなか。
これらはそもそも、動詞次第で似たり寄ったり。
戻り値もほぼプリミティブ型なのが難しいんですよ。

-->

---

# 本当に重要だったのは

- 名詞で使われている、attribute, cast, mutatorあたり
* あるいは、**Eloquent\Model本体から使われているメソッド**

<!--

理解した今から考えれば、名詞で使われている
attribute, cast, mutatorが重要、
そこから考えていけばよかった、とわかるんですが。

(めくる)
あるいは、こちらの方がいいかもしれません。
Eloquent\Model本体から使われているHasAttributesのメソッドはなにか、
という視点です。

-->

---

# Eloquent\Model本体から使われているメソッド

`getAttribute` `isDirty` `mergeAttributesFromCachedCasts` `setAttribute` `setRawAttributes` `syncChanges` `syncOriginal`

<!--

使われているメソッド全体はもうちょっと多いんですが、その中でも
複数回使われているものはこんな感じです。かなり絞られました。

-->

---

# Eloquent\Model本体から使われているメソッド

`getAttribute` `isDirty` `mergeAttributesFromCachedCasts` `setAttribute` `setRawAttributes` `syncChanges` `syncOriginal`

<style scoped>
code:nth-child(1),
code:nth-child(4),
code:nth-child(6),
code:nth-child(7) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

まあ割とどれも重要なんですが、具体的に使われている個所を見て、
特に重要に見えるのはこの辺ですね。

実際、getAttribute, setAttributeは
マジックメソッドから使用されているわけです。ちょっと考えれば、
インスタンスの使用時に頻繁に使われている個所だとわかります。

-->

---

```php
<?php

namespace Illuminate\Database\Eloquent\Concerns;

trait HasAttributes
{
    protected function transformModelValue($key, $value)
    {
        if ($this->hasGetMutator($key)) {
            return $this->mutateAttribute($key, $value);
        } elseif ($this->hasAttributeGetMutator($key)) {
            return $this->mutateAttributeMarkedAttribute($key, $value);
        }

        if ($this->hasCast($key)) {
            return $this->castAttribute($key, $value);
        }

        if ($value !== null
            && \in_array($key, $this->getDates(), false)) {
            return $this->asDateTime($value);
        }

        return $value;
    }
}
```

<!--

さて、重要な個所が見えてくると、
ほかのメソッドの立ち位置もはっきりしてきます。

getAttribute, setAttributeの実装をしっかり見ていくと、
アクセサ・ミューテタ、キャストなどの、
アトリビュートを変更する処理が、
アトリビュートの取得・設定をシンプルに
ラップしている形になっているのがわかります。

これはgetAttributeの奥で使われるメソッドですが、
ミューテタがあったら、キャストがあったら、日付だったら、
それぞれの処理が行われ、いずれでもない場合に単純に値を返す、
この場合これはアトリビュートのいずれかですね、という形です。

-->

---

- アトリビュートは、データベースの行の各カラムの値と対応
- アトリビュートは、取得・設定される際に、アクセサ・ミューテタ、キャストによって変更

<!--

アトリビュートは、データベースの行の各カラムです。
アトリビュートは取得・設定される際に、
アクセサ・ミューテタ、キャストなどによって変更されるのです。

HasAttributesは、アトリビュート自身だけでなく、
アクセサ・ミューテタ、キャストも含めた機能が
まとまっているトレイトです。

as, fromで始まるメソッドに関しても、やはり関連したものです。
ただこれでHasAttributesのすべてというわけではなく、
それ以外のものもいくらかあります。

-->

---

`append` `encryptUsing` `getArrayableAppends` `getArrayableItems` `getArrayableRelations` `getChanges` `getDateFormat` `getDates` `getDirty` `getOriginal` `getOriginalWithoutRewindingModel` `getRawOriginal` `getRelationValue` `getRelationshipFromMethod` `handleLazyLoadingViolation` `hasAppended` `hasChanges` `isClassDeviable` `isClassSerializable` `isClean` `isDirty` `isRelation` `isStandardDateFormat` `only` `originalIsEquivalent` `relationsToArray` `serializeDate` `setAppends` `setDateFormat` `syncChanges` `syncOriginal` `transformModelValue` `wasChanged`

<!--

HasAttributesのメソッドのうち、名前にattribute, cast, mutatorが
ついていない、またfrom, asで始まらない、ものがこちらになります。

-->

---

`append` `encryptUsing` `getArrayableAppends` `getArrayableItems` `getArrayableRelations` `getChanges` `getDateFormat` `getDates` `getDirty` `getOriginal` `getOriginalWithoutRewindingModel` `getRawOriginal` `getRelationValue` `getRelationshipFromMethod` `handleLazyLoadingViolation` `hasAppended` `hasChanges` `isClassDeviable` `isClassSerializable` `isClean` `isDirty` `isRelation` `isStandardDateFormat` `only` `originalIsEquivalent` `relationsToArray` `serializeDate` `setAppends` `setDateFormat` `syncChanges` `syncOriginal` `transformModelValue` `wasChanged`

<style scoped>
code:nth-child(6),
code:nth-child(9),
code:nth-child(10),
code:nth-child(11),
code:nth-child(12),
code:nth-child(17),
code:nth-child(21),
code:nth-child(25),
code:nth-child(30),
code:nth-child(31),
code:nth-child(33) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

名前を見ていくと、original, change(s), dirtyという名詞があります。

syncOriginalというメソッドが、Modelから直接使われている
数少ないメソッドの中に出てきたのを覚えていますか？
実はこの辺の機能は、行のライフサイクルに関するものです。

-->

---

# Eloquentのアトリビュートのライフサイクル

- 構成要素
  - 3つのプロパティ
    - `$attributes`
    - `$original`
       * 原本となるデータ
    - `$changes`
       * 前回変更した内容
  - いくつかのメソッド
    - `syncOriginal`
    - `syncChanges`

<!--

ライフサイクルについて説明します。
HasAttributesにある3つのプロパティ、$attributes, $original, $changes, 
さらにいくつかのメソッド、主にsyncOriginal, syncChanges,
この辺りが関係しているんですが、

(めくる)
$originalというのは、データベースにある原本のデータです。

(めくる)
$changesは、今度はデータベースに保存したときに、前回保存された
アトリビュートのうち、実際に変更があったものが入っています。
これはつまり、$attributesと$originalの差分で、この差分自体は、
dirtyという名前で呼ばれます。isDirtyとかありましたね。

-->

---

# Eloquentのアトリビュートのライフサイクル

<table>
  <tr>
    <th></th>
    <th>$attributes</th>
    <th>$original</th>
    <th>$changes</th>
  </tr>
  <tr>
    <th>1. 未保存の行</th>
    <td>あり</td>
    <td>なし</td>
    <td>なし</td>
  </tr>
  <tr>
    <th>2. 未保存から保存した行</th>
    <td>あり</td>
    <td>あり</td>
    <td>なし</td>
  </tr>
  <tr>
    <th>3. 取得した行</th>
    <td>あり</td>
    <td>あり</td>
    <td>なし</td>
  </tr>
  <tr>
    <th>4. 2, 3の状態から保存した行</th>
    <td>あり</td>
    <td>あり</td>
    <td>あり</td>
  </tr>
</table>

<style scoped>
table {
  margin-left: auto;
  margin-right: auto;
}
th {
  text-align: left;
}
</style>

<!--

syncOriginalは、データベースの原本を、
つまり取得したての$attributesの内容を、$originalに保存します。
原本があってこその$originalなので、たとえばnewで作ったインスタンスには
$originalは存在しません。

syncChangesは、データベースに保存したときに、
変更を$changesに保存します。しかしこれは、先程言ったように、
$originalと$attributesの差分です。なので新規での保存時には、
まだ$originalがないため、$changesもできません。

-->

---

# HasAttributesまとめ

- アトリビュートが中心
- アトリビュートを取り囲むようにアクセサ・ミューテタ、キャスト
- アトリビュートのライフサイクルとして、$original, $changes

<!--

HasAttributesは、アトリビュートを中心に、
アクセサ・ミューテタ、キャストによるアトリビュートの変更、
original, changesのようなライフサイクル、あたりを実装している個所、
と考えられそうです。

この辺の理解が深まると、Eloquent\Model本体も見えてきそうな感じですが、
その前にもう一つ、リレーションについて見てみましょう。
HasAttributes同様、Eloquent\Modelから使われている、
HasRelationshipsあたりです。

-->

---

# HasRelationships

- Eloquent\Modelから使われているトレイト
- HasAttributesに次ぐ規模
- とはいえメソッド数は50以下
- **リレーション関係の機能を持つ**

<!--

Eloquent\Modelから使われているトレイトの中で、
HasAttributesに次ぐ規模ではあるんですが、
それでもHasRelationshipsのメソッド数は50に届かないほどです。

そして、リレーションに関する機能が実装されています。
リレーションって割と重要な機能ですよね？
でもメソッド数は少ない。
じゃあ、また委譲とかあったりするのか、と思われるかもしれませんが、
今回はちょっと違います。


-->

---

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

<!--

さて、Eloquentでリレーションを使うときって、こんな感じですよね。
モデルクラスにhasOneとかhasMany, belongsToみたいなメソッドを
実行して返すメソッドを実装する。

-->

---

```php
<?php

namespace Illuminate\Database\Eloquent\Concerns;

use Illuminate\Database\Eloquent\Relations\HasOne;

trait HasRelationships
{
    public function hasOne($related, $foreignKey = null, $localKey = null)
    {
        // ...

        return $this->newHasOne($instance->newQuery(), $this, $instance->getTable().'.'.$foreignKey, $localKey);
    }

    protected function newHasOne(Builder $query, Model $parent, $foreignKey, $localKey)
    {
        return new HasOne($query, $parent, $foreignKey, $localKey);
    }
}
```

<!--

このhasOneやbelongsToのようなメソッド、これらがHasRelationshipsに
実装されているわけですが、そこでなにをしているのかというと、
それぞれメソッドごとに個別の、
Illuminate\Database\Eloquent\Relations\Relationを継承しているクラスの
インスタンスが生成されているんです。

-->

---

# リレーションに関係するクラス

- `BelongsTo`
- `BelongsToMany`
- `HasMany`
- `HasManyThrough`
- `HasOne`
- `HasOneThrough`
- `MorphMany`
- `MorphOne`
- `MorphTo`
- `MorphToMany`

<!--

ということで、ここでいきなり10クラスほど追加です。
HasRelationshipsと、そこで生成されるこの辺のクラス、
さらにEloquent\ModelにもEloquent\Builderにも、リレーションに関わる
機能はあちこちに実装されています。

さて、このクラス、一つずつ見ていったらきりがないので、
まとめて把握しましょう。

-->

---

# そもそもリレーションとは

<!--

さて、その前に、リレーションとはなんでしょう？
あるテーブルのある行の持つカラムを元に、
別のテーブルのある行を特定できるとき、
それらの行が関連している、という感じでしょうか。

-->

---

# そもそもリレーションとは

- users
  - **id**
  - ...
- posts
  - id
  - **user_id**
  - ...

<!--

具体的な実装としては、ちょっと分け方が難しいですが、
今回は3パターンあるとしましょう。

1つは、こちらのテーブルのあるキーの値を、
関連するテーブルのあるカラムが持っている、というパターンです。

Eloquentで言うと、hasOne, hasManyのパターンです。

-->

---

# そもそもリレーションとは

- posts
  - id
  - **user_id**
  - ...
- users
  - **id**
  - ...

<!--

2つ目は、1つ目のパターンの逆の視点です。
こちらのテーブルが、関連するテーブルのあるキーの値を持っている。

Eloquentで言う、belongsToです。

もしかするとEloquentを使い始めた当初、hasOne, hasManyはあるのに、
belongsToに対応するものがない、と思ったかもしれません。
belongsToManyはありますが、あれは多対多ですからね。

まあ当然で、こちらの視点からは、関連する行は1つになるからです。

-->

---

# そもそもリレーションとは

- users
  - **id**
  - ...
- role_user
  - **role_id**
  - *user_id*
- roles
  - *id*
  - ...

<!--

3つ目です。中間テーブルを介した多対多ですね。
こちらのテーブルと、関連するテーブルの両方のキーを、
中間テーブルが持っているという形です。

EloquentではbelongsToManyが該当します。

-->

---

- `BelongsTo`
- `BelongsToMany`
- `HasMany`
- `HasManyThrough`
- `HasOne`
- `HasOneThrough`
- `MorphMany`
- `MorphOne`
- `MorphTo`
- `MorphToMany`

<style scoped>
li:nth-child(1) code,
li:nth-child(2) code,
li:nth-child(3) code,
li:nth-child(5) code {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

さて、おかしいですね。
ここまでの3パターンで、Eloquentで該当するクラスは、まだ4つだけです。
HasOne, HasManyが1つ目、BelongsToが2つ目、BelongsToManyが3つ目。
残りの6つはなんでしょうか？

-->

---

- `BelongsTo`
- `BelongsToMany`
- `HasMany`
- `HasManyThrough`
- `HasOne`
- `HasOneThrough`
- `MorphMany`
- `MorphOne`
- `MorphTo`
- `MorphToMany`

<style scoped>
li:nth-child(4) code,
li:nth-child(6) code {
  color: #0d0;
  text-decoration: underline;
}
li:nth-child(7) code,
li:nth-child(8) code,
li:nth-child(9) code,
li:nth-child(10) code {
  color: #00d;
  text-decoration: underline;
}
</style>

<!--

よく見ると、名前にパターンがあります。
残っているものは、HasOne, HasManyにThroughがついたもの、
接頭辞にMorphがついたもの、の2パターンです。

-->

---

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    public function postComments()
    {
        return $this->hasManyThrough(Comment::class, Post::class);
    }
}
```

<!--

HasOneThrough, HasManyThroughはそれぞれ、
通常のリレーションと違い、
もう一段階先のテーブルにアクセスするリレーションですね。

たとえばUserがPostを投稿し、そのPostにCommentがつくような
リレーションで、Userから直接commentsを取得できるような感じです。

Morph系はポリモーフィック関連です。
SQLアンチパターンにあるやつですね。

-->

---

- `Eloquent\Relations\Relation` *
  - `HasOneOrMany` *
    - `HasOne`
    - `HasMany`
    - `MorphOneOrMany` *
      - `MorphOne`
      - `MorphMany`
  - `BelongsTo`
    - `MorphTo`
  - `BelongsToMany`
    - `MorphToMany`
  - `HasManyThrough`
    - `HasOneThrough`

<!--

さて、
これらのクラスの継承のツリーを書いてみると、面白いことがわかります。
その前に右に*がついているのは、これは抽象クラスですね。

このツリー構造を横目に、コードをざっと見てみます。

-->

---

- `Eloquent\Relations\Relation` *
  - <code class="highlight">HasOneOrMany</code> *
    - <code class="highlight">HasOne</code>
    - <code class="highlight">HasMany</code>
    - `MorphOneOrMany` *
      - `MorphOne`
      - `MorphMany`
  - `BelongsTo`
    - `MorphTo`
  - `BelongsToMany`
    - `MorphToMany`
  - `HasManyThrough`
    - `HasOneThrough`

<!--

SQLレベルで考えると、HasOneとHasManyは大差ありません。
なのでこの2つは、同じHasOneOrManyという抽象クラスを継承して、
それぞれ自身の実装は最小限です。

-->

---

- `Eloquent\Relations\Relation` *
  - `HasOneOrMany` *
    - `HasOne`
    - `HasMany`
    - <code class="highlight">MorphOneOrMany</code> *
      - <code class="highlight">MorphOne</code>
      - <code class="highlight">MorphMany</code>
  - `BelongsTo`
    - <code class="highlight">MorphTo</code>
  - `BelongsToMany`
    - <code class="highlight">MorphToMany</code>
  - `HasManyThrough`
    - <code class="highlight">HasOneThrough</code>

<!--

次に、ポリモーフィック関連のMorphなんとかクラスが、
元になるクラスと近くにあること。
そして独自に実装しているコードも、やはり最小限です。

ポリモーフィック関連は、Eloquentから見ると、
単に見るテーブルが動的に変わるだけで、
案外気にするところは少ないということではないでしょうか。

-->

---

- `Eloquent\Relations\Relation` *
  - <code class="highlight">HasOneOrMany</code> *
    - `HasOne`
    - `HasMany`
    - `MorphOneOrMany` *
      - `MorphOne`
      - `MorphMany`
  - `BelongsTo`
    - `MorphTo`
  - `BelongsToMany`
    - `MorphToMany`
  - <code class="highlight">HasManyThrough</code>
    - `HasOneThrough`

<!--

ほかにもHasOne, HasManyは親となる抽象クラスがあるのに、
HasOneThroughはHasManyThroughを直接継承しているのはなぜか、
みたいなところも気になりますが、時間もないのでこの辺は
みなさん考えてみてください。

Pivotなんかも面白いんですけどね、これも飛ばします。
一つだけ言うとこれはBelongsToManyでの中間テーブルなんですが、
Eloquent\Modelを継承しているんですよね。
つまり、やはりEloquent\Modelは特定のテーブルの抽象なんですよ。

-->

---

- <code class="highlight">Eloquent\Relations\Relation</code> *
  - `HasOneOrMany` *
    - `HasOne`
    - `HasMany`
    - `MorphOneOrMany` *
      - `MorphOne`
      - `MorphMany`
  - `BelongsTo`
    - `MorphTo`
  - `BelongsToMany`
    - `MorphToMany`
  - `HasManyThrough`
    - `HasOneThrough`

<!--

最後にもう1つだけ。Eloquent\Relations\Relationについてです。
リレーション関係の各クラスの基底クラスになっているこちらですが、
実はこれ、Eloquent\Builderに委譲しているクラスです。

BuilderContractというインターフェイスも実装していて、Eloquent\Builder
と等価と考えてよいものです。

Eloquent\Relations\Relationは、元となる行に関連する制約のついた
Eloquent\Builderと考えるとわかりやすいです。

-->

---

# Eloquentのリレーション

- Eloqeuntの中でも、大きな範囲を占める
- SQL/RDBMSのリレーションとは思想が違う(と思う)

<!--

さて、長くなりました。実際リレーションはほかにも、
hasやwithやload, その他いろいろあって興味深いところです。
Eloquentの中で、リレーションという機能はかなり大きく、広いものです。

なおSQL/RDBMSにももちろんリレーション自体はあるのですが、
Eloquentはそれをそのまま持つだけでなく、
オブジェクト指向に合わせた形にして持っています。

ある行から直接関連する行を取得できたり、ある行に関連するものとして、
直接別のテーブルの行を挿入できたり。

考え方としても、RDBMSでは関連する行を合成して一つの単位にして
扱いますが、Eloquentでリレーションを扱う際は、
リンクした別々のオブジェクトとして扱えます。

-->

---

# 

<!--

(4章)

HasAttributes, HasRelationshipsと、Eloquent\Modelから使われている
大きなトレイトを2つ見てきました。

残りはどれも小さなもので、理解も簡単です。HasTimestampsは、created_at,
updated_atあたりの関係ですね。

-->

---

# 

<!--

HasGlobalScopesも名前通り、グローバルスコープ関係です。
グローバルスコープは、モデル単位ですべてのクエリに制約をかけるといった
効果範囲の広い機能で、そのためEloquent\Model, Eloquent\Builderにもあちこちに
出てきますし、その上で単体のトレイトもあります。

論理削除を実現するSoftDeletesトレイトがこの機能を使って実装されてます。
それくらい広い範囲に影響する機能なので、ちょっと危険で、個人的には
あまり使いたくない機能ですが……。

-->

---

# 

<!--

HasEventsは、モデルの作成・更新・削除などの各タイミングで
フックを行うためのものです。

HideAttributesとGuardAttributesは入出力のセキュリティ機能です。
HideAttributesは$visible, $hiddenという2つのプロパティを持ち、
これらの設定によって、各種シリアライズ時に出力するかしないかを
決めます。

GuardAttributesはその逆に近いもので、
おなじみの$fillable, $guarded絡みですね。
これは入力を直接保存するような場合のガード用なのですが、
実際入力内容をバリデーションもせず保存する場合に、
カラム単位で保存の可否を制限できるだけで、本質的な安全性には無関係、
ですので私は使いません。
しかし使わないということも封じられているので、
空の$guardedを設定するのですが。
Eloquentの数多の機能の中でも、完全に間違った機能だと言い切れるのは
これだけです。

-->

---

# 

<!--

Eloquent\Model本体の機能も、ざっとですが見ていきましょう。

TODO: ここはスライドに頼るかな……。

クラスを使うときに常に一度だけ動く、boot的な処理だったり、
自身を含む関連オブジェクトの生成やらのファクトリ的なメソッド、
データベース接続関係、テーブル・カラム情報関係、リレーション関係、
シリアライズ関係、グローバルスコープ・ローカルスコープ、
ページネーション、オブジェクトとしての機能、
インターフェイスを実装するために必要なメソッド、……。

それぞれ必要な機能ではあるのですが、
Eloquentの全体像を知るために重要かというと。

やはりEloquent\Model本体に実装されている機能は、
雑多でトレイトにまとめづらいために残っているのかもしれません。

-->

---

# 

<!--

もうちょっとだけ続きます。次は、fill, save, updateやdeleteです。
fillは渡した値を$attributesに設定するメソッドです。
saveは、$attributesをデータベースに保存します。
updateはfillしてsaveですね。deleteは、単に行を削除します。

Eloquent\Modelには、クエリを直接操作するメソッドはほぼないのですが、
この辺は数少ない例外です。

なぜこういう例外があるのか。
単純に、updateやdeleteは、直接行に対しての処理だからです。

-->

---

# 

<!--

さて、ここで気になることがありますcreateはどこに行ったのでしょうか。
update, deleteがEloquent\Modelにあるのに、createはありません。
createはEloquent\Builderにあります。
ついでに言うと、Query\Builderにもありません。
Query\Builderにはinsertはありますが、createはなく、
Eloquent\Builderにはcreateはありますが、insertはありません。

冒頭、User::insertのようにした場合の話をしましたが、
もうわかったと思います。User::insertはQuery\Builder::insertを呼んでいて、
Query\BuilderはEloquent\Builderとは違い、テーブルに関する情報を持っていません。
そのため、タイムスタンプには関知しない、というわけでした。

-->

---

# 

<!--

戻ります。createが、Eloquent\Modelではなく、Eloquent\Builderにあるのは、
一体どういう意味を持つのでしょうか？
もうわかりますよね。createは行に対する処理ではないからです。
SQLでINSERTするとき、テーブル名は指定しますが、
WHERE句で行を指定するわけではありません。
もちろんSELECTしたものをINSERTする場合は別ですが。

-->

---

# 

<!--

この辺でもう一つ見えてこないでしょうか？
Eloquent\Modelを継承したクラスは、クラスとして使用するとき、
たとえば静的メソッドを実行したり、プロパティや定数、
あるいはメソッドを定義するとき、
この場合には、テーブルを表現しています。

-->

---

# 

<!--

一方、インスタンスとして生成された場合には、今度は行を表現しています。
気付いてしまえば簡単でわかりやすいことですが、
私は使い始めてから、しばらくの間気付きませんでした。

-->

---

# 

<!--

この使い分けを理解していれば、たとえばcreateがEloquent\Modelではなく
Eloquent\Builderに実装されていることも自然に感るのではないでしょうか。。

-->

---

# 

<!--

またたとえば、私は昔$userインスタンスからwhereが生えてるというコードを
見たことがあるんですが、
当時、このようなコードに対して適切な指摘はできませんでした。

これはおかしいですが、実際問題なく動きます。

ドキュメントにそういう書き方はしていない、普通そう使わない、
あるいはせいぜい、インスタンスである意味がない、くらいでしょうか、
そういうコードをレビューするとしても、それくらいのことしか
言えなかったと思います。

しかしモデルがクラスであるか、インスタンスであるかによって
テーブルの表現だったり行の表現だったりすることを知った今では、
「行の表現であるインスタンスからwhereを呼び出すのはおかしい」
と言えるわけです。

-->

---

# 

<!--

まとめです。
Eloquentの中心となるのは、Eloquent\Modelを継承したクラスです。
たとえばQuery\BuilderがSQLクエリと、
その実行を全面的に表現しているのに対して、
モデルクラスが表現するのは特定のテーブルと、その行です。

クラスとして使用する場合はテーブルを、インスタンスとして使用する場合は、
さらにテーブル内の特定の行を表現していると考えてよさそうです。

-->

---

# 

<!--

実装においては、特にテーブルを基準に行われるクエリ処理は、
Eloquent\Builder, さらにQuery\Builderへの委譲によって実装されています。
行が持つカラムは、アトリビュートとして、
Eloquent\Modelが使用しているHasAttributesを中心に実装されています。

-->

---

# 

<!--

特定のテーブルを表現しているモデルクラスは、
テーブルごとの知識を持つことができます。

主キーの名前や型、タイムスタンプの名前や型、それによって、
これらのキーを使うときにただ既定の名前を想定するのではなく、
フレキシブルに実行できます。

また、その他のカラムについても型の知識を持つことができ、
それによってデータベースのカラムから、
行インスタンスのプロパティとしてアクセスする際に、
アクセサ・ミューテタ、キャストを行うことで、
適切な型に変換することができます。

-->

---

# 

<!--

テーブルごとの知識は、モデルクラスがプロパティとして持ったり、
定数やメソッドとして持つこともあります。
これらの実装はModel本体やHasTimestampsに。
アクセサ・ミューテタ、キャストはこれもHasAttributesに。

-->

---

# 

<!--

行からは行自体の操作や、関連するほかのテーブル、
つまりモデルの行の操作を行うこともできます。

前者はupdate, delete, 後者はリレーション。
前者はE\M本体に、後者はHasRelationships経由で、
各リレーション系のクラスに実装されています。

-->

---

# 

<!--

Eloquentは全体的に、SQL/RDBMSを表現していると言えますが、
それらの機能をあきらかに超えている部分があります。

リレーションがそうですね。データベースの方では、ある行自体から、
関連する別の行へのリンクを持つ形ではないので。
データベースはむしろ、関連する行を、一つの行として表現します。

-->

---

# 

<!--

その他で大きいのはグローバルスコープです。
データベースのビューに近いイメージもありますが、
より柔軟で、扱いづらいものだと思います。

逆にローカルスコープは、実装的にはごく小さな機能ですが、
SQLの最もやっかいな、WHERE句の条件の再利用ができないという点を
非常にシンプルに解決していてとても強力な機能だと思っています。

-->

---

# 

<!--

細かい機能もいろいろあります。行をカラム単位で変更して、
行単位で更新・削除したり、そのような処理の流れのライフサイクルがあり、
ライフサイクルごとにイベントを設定できたりします。

-->

---

# 

<!--

単純にオブジェクトとしての有効性のための、
同じ行かどうかの比較メソッドがあったり、行の複製ができたりも。

-->

---

# 

<!--

Laravelのほかの機能や、その他外部との連携を考えた機能もあります。
前者は、ルートとのバインド、ページネーションや、ブロードキャスト、
あたりとの連携、後者はシリアライズや入出力セキュリティでしょうか。

-->

---

# 

<!--

以上が私の考えるEloquentの全貌です。巨大ですが、
基本的にはデータベースの特定のテーブル・行の割と素直な表現が中心です。

いくつかの、SQL/RDBMSにはない機能と、
その他の細かな関連機能がついてはきていますが、
大きさの割には構成要素はシンプルだと思います。

というか、シンプルに考えられるようにまとめたから、でしょうが。

TODO: 最後いる？

-->

---

# 

<!--

TODO: 終わりのあいさつ。

-->

