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

(1章: 0分～)(6分)

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

# $fillableは具体的にどこで有効か

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

# createの場合は、setterでの代入の場合は

```php
<?php

// createの場合は？
User::create([
    'name'              => 'tarou',
    'email'             => 'tarou@example.com',
    'password'          => bcrypt('password'),
    'email_verified_at' => now()
]);

// setterの場合は？
$user = new User();
$user->name              = 'tarou';
$user->email             = 'tarou@example.com';
$user->password          = bcrypt('password');
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

# firstOrCreateの場合は

```php
<?php

$user = User::firstOrCreate([
    'email'             => 'tarou@example.com',
], [
    'name'              => 'tarou',
    'password'          => bcrypt('password'),
    'email_verified_at' => now(),
]);
```

<!--

たとえば、firstOrCreateの第2引数で使う場合はどうでしょう？

この例などは、表面的に知っているだけでは難しそうです。

-->

---

# このリレーションクエリはなにがおかしい

```php
<?php

$user->posts()->updateOrCreate(['user_id' => $user->id], $attributes);
```

<!--

私は昔、このようなコードを書いてしまったことがあります。
このコードのどこがおかしいか、すぐにわかりますか？
詳しい人であれば、私がなぜ間違ったのか、
その理由さえ一目見ただけでわかるかもしれません。

ですがちょっとコードを追ったくらいではわからない人もいるはずです。
実際私はそうでした。

-->

---

# insertでcreated_at, updated_atが入らないのはなぜか

```php
<?php

use App\Models\User;

User::insert([
  ['name' => 'tarou', 'email' => 'tarou@example.com', 'password' => bcrypt('password')],
  ['name' => 'jirou', 'email' => 'jirou@example.com', 'password' => bcrypt('password')],
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
$user  = User::where('id', 1)->first();

echo $user->name;
$user->update(['password' => bcrypt('password')]);
```

<!--

(2章: 6分～)(16→10分？)

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

### whereはEloquent\Builderに実装されているものを委譲で呼び出している

User::where
→ User::__callStatic
→ User::__call
→ Eloquent\Builder::where

<!--

whereは、Eloquent\Builderというクラスに実装されています。
そちらに処理を委譲している形です。

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

# ここまでのメソッド数

- Eloquent\Model(を継承したクラス) ... 350メソッド
- Eloquent\Builder ... 162メソッド
- 計 ... 512メソッド

<!--

Eloquent\Modelと合わせて、メソッドが500を超えています。
とはいえこれで一通り、よく使うメソッドは揃ったかな、
と思いきや、実はまだ足りないものがあります。

-->

---

## Eloquent\ModelにもEloquent\Builderにもないメソッド

- `select`
- `join`
- `groupBy`
- `orderBy`
- etc...

<!--

お気付きになられたでしょうか？
selectも、joinも、groupByもorderByも、今の一覧にはありませんでした。

どうなっているのでしょう？　まあ、予想つきますよね。

-->

---

### Eloquent\BuilderにもないメソッドはQuery\Builderのものを呼び出している

Eloquent\Builder::select
→ Eloquent\Builder::__call
→ Illuminate\Database\Query\Builder::select

<!--

Eloquent\Builderからは、さらに別のクラスのメソッドが
呼び出されるようになっています。
Illuminate\Database\Query\Builderというクラスです。

Eloquent\ModelからEloquent\Builderへの委譲同様、
マジックメソッドによって、Eloquent\Builderにないメソッドは
Query\Builderのものが呼び出されるようになっています。

先程出てきたselect, joinなどなど。
そして今度こそ、私たちが普段使っている機能が一通り揃いました。

さすがにしつこいのでメソッドの一覧はもう出しませんが、
Query\Builderには200を超えるメソッドがあり、
select, join, groupBy, orderByほか、普段Eloquentで
SQLを操作する際に使うメソッドは、ほとんどここに実装されています。

-->

---

# User::whereが返すのはEloquent\Builder

```php
<?php

echo get_class(User::where('id', 1));
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

# Query\Builderとは

- 200を超えるメソッドの半数以上が、自身を返すメソッド
  - 大半の機能がメソッドチェーンでSQLを組み立てるもの
- 残りの多くは、取得した行、そのコレクションを返す
- **Query\Builderは、メソッドチェーンでSQLクエリを表現し、それを実行するもの**

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
  - Query\Builderは、Illuminate\Database\\**Query\Builder**

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
しているということは、そこになにか、意味がありそうです。

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`
  - etc...

<!--

Query\Builderをオーバーライドしているもの。
まあいろいろあるんですが、目立つのは、whereやfind, get,
あとはupdate, deleteあたりですかね。

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
  - etc...

<style scoped>
li li:nth-child(1) code,
li li:nth-child(4) code,
li li:nth-child(5) code {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

理由はいくつかあります。
一つは、グローバルスコープをQuery\Builderに適用したいというものです。
(where, update, delete, increment, decrement)

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`
  - etc...

<style scoped>
li li:nth-child(2) code {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

もう一つは、戻り値を変更したいものです。
Query\Builderであれば、stdClassや、そのコレクションを返すところを、
モデルのインスタンスや、そのEloquentのコレクションを返したい。
またリレーションに関する処理が追加される場合もあります。
(find, get)

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`
  - etc...

<style scoped>
li li:nth-child(4) code:nth-child(1),
li li:nth-child(5) code {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

updated_atの処理をしたい、というものもあります。
(update, increment, delete)

-->

---

# Eloquent\Builderのメソッド

- Query\Builderをオーバーライドしているもの
  - `where`
  - `find` `get`
  - `latest` `oldest`
  - `update` `delete`
  - `increment` `decrement`
  - etc...

<style scoped>
li li:nth-child(4) code:nth-child(2) {
  color: #f00;
  text-decoration: underline;
}
</style>

<!--

例外的なのがdeleteで、これはonDeleteという、デフォルトでは
SoftDeletesのためにしか使われていない特殊な機能のために、
独自に実装されています。
(delete)

-->

---

# Eloquent\Builderの独自のメソッド

- Eloquent\Builderだけにあるもの
  - 名前に`key`とか`timestamp`を含むもの
    - `whereKey` `whereKeyNot`
  - 名前に`scope`を含むもの
    - `withGlobalScope` `withoutGlobalScope` `callNamedScope`
  - 名前に`eagerLoad` `relation` `with`を含むもの
    - `eagerLoadRelations` `getRelation` `with` `without`
  - etc...

<!--

Eloquent\Builderにのみ実装されている、
独自のメソッドについても見てみましょう。名前で分類してみます。

whereKeyはfindから呼び出されているメソッドで、
Query\Builderでは決め打ちでidというカラムを使うところで、
実際の主キーの名前をちゃんと見て、処理するようにしています。

scopeが含まれるものは、見ての通りグローバルスコープ、
ローカルスコープ関係です。

eagerLoad, relation, withなどを含むものは、リレーション関係ですね。

ということで、独自のメソッドも、オーバーライドされているメソッドも、
存在理由はどちらも似たり寄ったりなのかなと思います。

-->

---

# なぜ直接Query\Builderを使わないのか

- Eloquent特有の機能のため
  - リレーション
  - グローバルスコープ
  - テーブルの情報
  - Eloquent\BuilderはQuery\Builderにこれらの機能を含めたサブクラス的な存在なのかも

<!--

ということで、Eloquent\ModelがQuery\Builderを直接使わず、
Eloquent\Builder経由で使う理由はなんとなくわかったかと思います。

Eloquent特有の機能、リレーションやグローバルスコープ、
あるいはテーブルの情報、主キーやタイムスタンプのカラム名ですね、
これらを持っていること、その辺に関連した機能があり、
単にQuery\Builderを直接使うだけでは不十分、ということでしょう。

// ここ、Eloquent\Modelにもクエリ関係のメソッドはあるので、不十分……。

// TODO: BuildQueries, 余裕あればこれについても書きたい。

/*
- 共通で使用しているBuildQueriesトレイトにあるもの
  - chunk, each, lazy
  - first
  - paginator, simplePaginator
  */

-->

---

# ここまでにわかったこと

* Eloquentの機能のうち、少なくない部分がEloquent\Builder, Query\Builderにある
  * Eloquentの外部のものであるQuery\Builderに、Eloquent特有の機能のための対応を加えた形のEloquent\Builder
* Builder 2つを合わせたメソッド数は、Eloquent\Modelのメソッド数に匹敵
  * Eloquentの半分は、クエリビルダでできている？

<!--

ここまででわかったことを簡単にまとめます。

(めくる)
Eloquentの機能のうち、少なくない部分がEloquent\Builder,
Query\Builderによるものでした。
SQLを組み立てて、実行して、というあたりはほぼそうです。

(めくる)
Query\BuilderにEloquent特有の機能のためのあれこれを加えたのが、
Eloquent\Builder.

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

# 閑話休題

```php
<?php

User::where('id', 1)        // これはEloquent\Builderにある
    ->select('id', 'email') // これはQuery\Builderにしかないはず？
    ->get();                // これはEloquent\Builderにある
```

<!--

さて、ちょっと話は変わるのですが。

Eloquent\Builderをチェーンしつつ
Query\Builderのメソッドを使えるのって、
最初どういう実装になっているのかわからなかったんです。
みなさんはこれ、わかりますか？

Eloquent\BuilderにあるwhereがEloquent\Builderを返す、
これは$this返せばいいだけです、問題ありません。

最後のgetも、Eloquent\Builderにありますから、問題ないですよね。
でも間のselect, これ、Query\Builderにしかないんです。
では$thisを返しても、Query\Builderが返って来るはずでは……？

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

実はこんな感じになってました。

単純に、委譲した結果をそのまま返すのではなく、
返すのはEloquent\Builder自身になっているんです。
Query\Builderに委譲した分の戻り値は無視です。

面白い実装ですが、こうなると気になってくるのが、
チェーンするメソッド以外の場合、つまり自身以外の値を返すものです。
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

同じ__callマジックメソッド内、
ちょっと上のコードに答えがありました。省略していた部分です。
forwardCallToの前のブロックですね。

$passthruで指定されたメソッド、これらに委譲する場合、
Eloquent\Builderを返すのではなく戻り値自体を返すようになっています。
単純に、戻り値が必要なメソッドは手動でリストアップして、
特別扱いしているんですね。

よく工夫されています。その分、理解するのは大変ですが。

-->

---

# 閑話休題 回答編

```php
<?php

User::where('id', 1)
    // Query\Builderにしかないメソッド。でもEloquent\Builderから実行する場合は
    // Query\Builder::selectの戻り値は無視して、Eloquent\Builderを返している
    ->select('id', 'email')
    ->get();

User::where('id', 1)
    // これもQuery\Builderにしかないメソッド。でも
    // Eloquent\Builder::$passthruに設定されているので、戻り値をそのまま返す
    ->count();
```

<!--

ということで、こういうことでした。

Query\BuilderにしかないメソッドをEloquent\Builderから実行するとき、
基本的には戻り値を無視してEloquent\Builderを返します。

が、$passthruに設定されているメソッドの場合は、戻り値をそのまま
返します。

-->

---

# ここまでのお話

- Eloquent
  - SQLクエリを組み立てて実行する部分
  * **SQLクエリを実行して取得した結果を使う部分？**

<!--

(3-1章: 22分～)(21分？)

SQLクエリを組み立てて、実行するというあたりがEloquentの大きな部分
と考えてよさそう、というところまでお話しました。

(めくる)
では次は、クエリを実行して取得したその結果、
それを使っていく方を見てみましょう。

-->

---

# 取得したインスタンスの型

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
まず型を見てみましょう。tinkerであれこれ実行してみます。

これは当然、Eloquent\Modelを継承したクラスのインスタンスです。
デフォルトのUserクラスは、ちょっと間に一つ挟んでるのですが。

-->

---

# データベースのカラム＝Eloquentのアトリビュート

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
    use Concerns\HasAttributes;

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

__get, __setの中身はEloquent\Modelが使用しているHasAttributes
トレイトのgetAttribute, setAttributeです。

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

なぜか。どの辺のメソッドが、HasAttributesの中心になるかが、
ぱっとはわからなかったからです。

(めくる)
get, set, is, has, あとas, fromみたいな、
同じような動詞で始まるメソッドばかりだったり。

(めくる)
これらの対象となる名詞もまた、同じようなものがたくさん。
attribute, cast, mutator, ...

(めくる)
引数や戻り値で見てもなかなか。
これらはそもそも、動詞次第で似たり寄ったり。
isやhasはboolean返って来るに決まってますからね。

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

# getAttributeの奥の方で行われていること

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
という分岐で、
それぞれの処理が行われ、いずれでもない場合、値をそのまま返す、
この場合これはアトリビュートのいずれかですね、という形です。

-->

---

# アトリビュートとは

- アトリビュートは、データベースの行の各カラムの値と対応
- アトリビュートは、取得・設定される際に、アクセサ・ミューテタ、キャストによって変更

<!--

アトリビュートは、データベースの行の各カラムです。
アトリビュートは取得・設定される際に、
アクセサ・ミューテタ、キャストなどによって変更されるのです。

HasAttributesは、アトリビュート自身だけでなく、
アクセサ・ミューテタ、キャストも含めた機能が
まとまっているトレイトでした。

as, fromで始まるメソッドに関しても、やはり関連したものです。
asはある型に変換する、fromはある型から変換する。

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
名前そのままですね。

(めくる)
$changesは、今度はデータベースに保存したときに、前回保存された
アトリビュートのうち、実際に変更があったものが入っています。
ちょっとわかりづらいですが、
これはつまり、$attributesと$originalの差分です。

保存時に、実際に変わった値だけが、$changesに保管される、
という感じです。

この差分自体は、dirtyという名前で呼ばれます。
isDirtyとかありましたね。

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

そしてメソッドです。

syncOriginalは、データベースの原本を、
つまり取得したての$attributesの内容を、$originalに保存します。
原本があってこその$originalなので、たとえばnewで作ったインスタンスには
$originalは存在しません。

syncChangesは、データベースに保存したときに、
変更を$changesに保存します。しかしこれは、先程言ったように、
$originalと$attributesの差分です。なので新規での保存時には、
まだ$originalがないため、$changesもできません。

言葉で説明してもさっぱりですよね。表を見てください。
2, 3は同等です。
データベースに存在する行なら、$originalがあり、
今回取得してから実際に変更がある更新をしているなら、
$changesがある、という感じです。

-->

---

# HasAttributesまとめ

- アトリビュートが中心
- アトリビュートを取り囲むようにアクセサ・ミューテタ、キャスト
- アトリビュートのライフサイクルとして、$original, $changes

<!--

まとめです。

HasAttributesは、アトリビュートを中心に、
アクセサ・ミューテタ、キャストによるアトリビュートの変更、
original, changesのようなライフサイクル、あたりを実装している個所、
と考えられそうです。

この辺の理解が深まると、Eloquent\Model本体も見えてきそうな感じです
が、その前にもう一つ、リレーションについて見てみましょう。
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

(3-2章: 34分～???)(10分？)

Eloquent\Modelから使われているトレイトの中で、
HasAttributesに次ぐ規模ではあるんですが、
それでもHasRelationshipsのメソッド数は50に届かないほどです。

名前通り、リレーションに関する機能が実装されています。
リレーションって割と重要な機能ですよね？
でもメソッド数は少ない。
じゃあ、また委譲とかあったりするのか、と思われるかもしれませんが、
今回はちょっと違います。

-->

---

# リレーションの基本的な使い方

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

<!--

さて、Eloquentでリレーションを使うときって、こんな感じですよね。
モデルクラスにhasOneとかhasMany, belongsToみたいな
メソッドを実行して返すメソッドを実装する。

-->

---

# hasOne, hasManyのようなメソッドがしていること

```php
<?php

namespace Illuminate\Database\Eloquent\Concerns;

// Illuminate\Database\Eloquent\Relations\Relationを継承しているクラス
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
それぞれメソッドごとに個別のクラスのインスタンスを生成しています。

そして、これらのクラスが、共通して
Illuminate\Database\Eloquent\Relations\Relationを継承しています。

-->

---

# リレーションに関係するクラス

- すべてEloquent\Relations\Relationを直接・間接に継承している
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
リレーションに関わる機能は、
HasRelationshipsと、そこで生成されるこの辺のクラス、
さらにEloquent\ModelにもEloquent\Builderにも、実装されています。

さて、この10クラス、一つずつ見ていったらきりがないので、
まとめて把握しましょう。

-->

---

# そもそもリレーションとは

<!--

その前に、リレーションとはなんでしょう？
あるテーブルのある行の持つカラムを元に、
別のテーブルのある行を特定できるとき、
それらの行が関連している、という感じでしょうか。

-->

---

# そもそもリレーションとは: パターン1

親テーブルのキーの値を、関連するテーブルが持つ

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

1つは、親テーブルのあるキーの値を、
関連するテーブルのあるカラムが持っている、というパターンです。

Eloquentで言うと、hasOne, hasManyのパターンです。

-->

---

# そもそもリレーションとは: パターン2

親テーブルが、関連するテーブルのキーの値を持つ

- posts
  - id
  - **user_id**
  - ...
- users
  - **id**
  - ...

<!--

2つ目は、1つ目のパターンの逆の視点です。
親テーブルが、関連するテーブルのあるキーの値を持っている。

Eloquentで言う、belongsToです。

もしかするとEloquentを使い始めた当初、hasOne, hasManyはあるのに、
belongsToに対応するものがない、と思ったかもしれません。
belongsToManyはありますが、あれは多対多ですからね。

まあ当然で、親の視点からは、関連する行は1つになるからです。

-->

---

# そもそもリレーションとは: パターン3

中間テーブルが、親と関連するテーブルの両方のキーを持つ

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
親テーブルと、関連するテーブルの両方のキーを、
中間テーブルが持っているという形です。

EloquentではbelongsToManyが該当します。

-->

---

# リレーションのパターンと対応するクラス

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

# それ以外のクラス

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

# HasOneThrough, HasManyThrough

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

またMorph系はポリモーフィック関連です。
SQLアンチパターンにあるやつですね。

-->

---

### Eloquent\Relations\Relationからの継承のツリー

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

さて、これらのクラスの継承のツリーを書いてみます。
なお右に*がついているのは、これは抽象クラスですね。

このツリー構造を抑えた上で、コードを見ていくと
面白いことがわかります。

-->

---

### HasOne, HasManyは大差ない

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
そうですよね。その気になればテーブルの構造は同じまま、
HasOneにしたりHasManyにできますから。

なのでこの2つは、同じHasOneOrManyという抽象クラスを継承して、
それぞれ自身の実装は最小限です。

-->

---

### ポリモーフィック関連のクラスは、元となるクラスを少し拡張しただけ

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

### HasOneOrManyが個別に存在するのに、HasOneOrManyThrough相当はない

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

このツリーにもないですが、
Pivotなんかも面白いんですけどね、これも飛ばします。
一つだけ言うとこれはBelongsToManyでの中間テーブルなんですが、
Eloquent\Modelを継承しているんですよね。
つまり、やはりEloquent\Modelは特定のテーブルの抽象なんですよ。

-->

---

### Eloquent\Relations\RelationはEloquent\Builderと同等

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

リレーションクラスでwhereとかが使えるのは、そのためです。

また、Illuminate\Contracts\Database\Eloquent\Builderという
インターフェイスが存在するのですが、
これを実装しているのって、Eloquent\Builderと、
Eloquent\Relations\Relationだけなんですよね。
この2つのクラスは、兄弟みたいな関係と考えてよさそうです。

Eloquent\Relations\Relationは、元となるリレーションメソッドに
関連する制約のついたEloquent\Builderと考えるとわかりやすいです。

-->

---

# Eloquentのリレーション

- Eloquentの中でも、大きな範囲を占める
- SQL/RDBMSのリレーションとは思想が違う(と思う)

<!--

さて、長くなりました。実際リレーションはほかにも、
hasやwithやload, その他いろいろあって興味深いところです。
Eloquentの中で、リレーションという機能はかなり大きく、広いものです。

なおSQL/RDBMSにももちろんリレーション自体はあるのですが、
Eloquentはそれをそのまま持つだけでなく、
オブジェクト指向に合わせた形にして持っています。
だいぶアレンジされている感じです。

ある行から直接関連する行を取得できたり、ある行に関連するものとして、
直接別のテーブルの行を挿入できたり。

考え方としても、RDBMSでは関連する行を合成して一つの単位にして
扱いますが、Eloquentでリレーションを扱う際は、
リンクした別々のオブジェクトとして扱えます。

-->

---

# Eloquent\Modelが使用するその他のトレイト

```php
<?php

namespace Illuminate\Database\Eloquent;

abstract class Model
{
    use Concerns\HasAttributes,
        Concerns\HasEvents,
        Concerns\HasGlobalScopes,
        Concerns\HasRelationships,
        Concerns\HasTimestamps,
        Concerns\HidesAttributes,
        Concerns\GuardsAttributes,
        ForwardsCalls;

    // ...
}
```

<!--

(4-1章: 43分～???)(10分？)

HasAttributes, HasRelationshipsと、Eloquent\Modelから使われている
大きなトレイトを2つ見てきました。残りのトレイトも見てみましょう。
残りはいずれも小さなもので、理解も簡単です。

-->

---

# HasEvents

```php
<?php

namespace App\Models;

use App\Events\UserCreated;
use App\Events\UserRetrieved;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    protected $dispatchesEvents = [
        'created' => UserCreated::class,
        'retrieved' => UserRetrieved::class,
    ];
}
```

<!--

HasEventsは、イベント機能を実装するトレイトです。
モデルのライフサイクルのいろいろなタイミングにフックできるようにします。

-->

---

# HasGlobalScopes

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    protected static function booted()
    {
        static::addGlobalScope('available', function (Builder $builder) {
            $builder->where('is_available', 1);
        });
    }
}
```

<!--

グローバルスコープは、
モデル単位ですべてのクエリに制約をかける機能です。

影響範囲が広いため、関連する実装はあちこちに散らばっています。
Eloquent\Model, Eloquent\Builder, そして当然このトレイトです。

SoftDeletesはこの機能を使って実現されています。

-->

---

# HasTimestamps

```tinker
>>> $user = User::create([ 'name' => 'tarou', 'email' => 'tarou@example.com', 'password' => bcrypt('password')]);
=> App\Models\User {#4530
     name: "tarou",
     email: "tarou@example.com",
     #password: "$2y$10$dP4QYy/dpXW.iG4ZhuCuMeCm987b9xtXhMFpHceV2eBf8t/yQcTtu",
     updated_at: "2022-08-18 08:49:29",
     created_at: "2022-08-18 08:49:29",
     id: 1,
   }
```

<!--

タイムスタンプは、デフォルトではcreated_at, updated_atという名前のカラムに、
自動で作成日時・更新日時を設定する機能です。

-->

---

# HidesAttributes

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    protected $hidden = [
        'password',
        'remember_token',
    ];
}
```

<!--

HidesAttributesは、行をシリアライズする際に、見せるもの、隠すものを
指定する機能です。

LaravelでJSONを返すのってとても簡単で、いちばん簡単なのはコントローラの
戻り値にモデルを指定するだけです。

このとき、素直にすべてのアトリビュートを出力してしまっては危険です。
そのような場合に、$visible, $hiddenで出力していいもの、駄目なものを
指定できます。

-->

---

# GuardsAttributes

```php
<?php

namespace App\Models;

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

GuardsAttributesはHidesAttributesの逆、
HidesAttributesが出力するデータの保護だったのに対し、
GuardsAttributesは入力するデータの保護のためのものです。
$fillable, $guardedプロパティで指定します。

具体的には、fillやfillを使うメソッドでの一括代入を可能にする
アトリビュートを制限するものです。
バリデーションもせず$request->allでそのまま突っ込むような場合ですね。

ただこちら、有効性は非常に限定されているのであまり意味はないです。

ということで、残りのトレイトの説明でした。
HasAttributesやHasRelationshipsと違い、Eloquentの全体像に影響のある
感じではないですね。

-->

---

# Eloquent\Model本体に実装されている機能

- クラスを使うときに常に一度だけ動く、boot的な処理
- 自身を含む関連オブジェクトを生成するファクトリ的なメソッド
- データベース接続関係
- テーブル・カラム情報関係
- リレーション関係
- シリアライズ関係
- グローバルスコープ・ローカルスコープ関係
- ページネーション関係
- オブジェクトとしての有用性のための機能
- インターフェイスを実装するために必要なメソッド

<!--

Eloquent\Model本体の機能も、ざっとですが見ていきましょう。

クラスを使うときに常に一度だけ動く、boot的な処理だったり、
自身を含む関連オブジェクトの生成やらのファクトリ的なメソッド、
データベース接続関係、テーブル・カラム情報関係、リレーション関係、
シリアライズ関係、グローバルスコープ・ローカルスコープ、
ページネーション、オブジェクトとしての機能、
インターフェイスを実装するために必要なメソッド、……。

トレイトにもあった機能の関係だったり、それ以外だった。
それぞれ必要な機能ではあるのですが、
Eloquentの全体像を知るために重要かというと。

やはりEloquent\Model本体に実装されている機能は、
雑多でトレイトにまとめづらいために残っているのかもしれません。

-->

---

# fill, save, update, delete

```php
<?php

// 以下2行はほぼ同等
$user->fill(['name' => 'saburou'])->save();
$user->update(['name' => 'saburou']);

$user->delete();
```

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

# createはどこ？

- update, deleteはEloquent\Modelにある
* createは、**Eloquent\Builderにある**

<!--

さて、ここで気になることがありますcreateはどこに行ったのでしょうか。
update, deleteがEloquent\Modelにあるのに、createはありません。

(めくる)
createはEloquent\Builderにあります。

-->

---

# create, update, deleteの使い方

```php
<?php

$user = User::create(...);
$user->update(...);
$user->delete();
```

<!--

create, update, deleteの使い方をそれぞれ見れば、
納得行くのではないでしょうか。

createは、update, deleteとは違い行に対する処理ではないからです。
テーブルに対する処理です。テーブルに対する処理はEloquent\Builderに
実装されて、モデルクラスの静的メソッドとして呼ばれます。

SQLでINSERTするとき、テーブル名は指定しますが、
WHERE句で行を指定するわけではありませんよね。

-->

---

# モデルをクラスの場合とインスタンスの場合とで、<br>表現しているものが違う？

<!--

updateやdeleteのように、行を対象とした操作であれば、クエリ操作でも
Eloquent\Modelに実装されることがあるということのようです。

それを考えると、
Eloquent\Builderがクエリ操作、Eloquent\Modelが行の操作、
という考え方は、なにか足りない気がします。

むしろ、モデルをクラスとして扱うときにはテーブルを表現していて、
インスタンスとして扱うときには行を表現している、
と考える方が適切ではないでしょうか。

私はLaravelを使い始めた当初、Eloquent\Modelを継承したクラスが、
テーブルの抽象のようでもあり、行の抽象のようでもある、ということに
しばらく混乱していました。

しかしあるとき、クラスとして扱うときはテーブルで、
インスタンスとして扱うときは行だ、と気付き、とてもしっくりきました。

-->

---

### モデルクラスは、クラスとして使用するときテーブルを表現している

```php
<?php

// usersテーブルから、id = 1の行を取得
User::find(1);

// usersテーブルから、email like '%@example.com'の行をすべて取得
User::where('email', 'like', '%@example.com')->get();

// another_db上のusersテーブルの行をすべて取得
User::on('another_db')->all();
```

<!--

モデルクラスは、クラスとして使用するときテーブルを表現しています。
たとえば静的メソッドを実行したり、

-->

---

### モデルクラスは、クラスとして使用するときテーブルを表現している

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    // usersテーブルのemailカラムにデフォルト値を設定
    protected $attributes = ['email' => 'dummy@example.com'];

    // シリアライズするときに表示しないカラムを指定
    protected $hidden = ['password', 'remember_token'];

    // 作成日時のタイムスタンプとして扱うカラムを指定
    const CREATED_AT = 'create_datetime';

    // postsテーブルへのリレーションを設定
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

<!--

プロパティや定数、あるいはリレーションを定義するとき、
この場合には、テーブルを表現しています。

-->

---

# インスタンスとして使用するときは、行を表現している

```php
<?php

$user = User::find(1);
echo $user->name;

foreach ($user->posts as $post) {
    // ...
}

$user->update([...]);
```

<!--

一方、インスタンスとして生成された場合には、今度は行を表現しています。
一度そう考えると、ほかの考え方はできないくらいしっくりくるんですが、
私はこの考え方に結構長い間気付きませんでした。

-->

---

# モデルインスタンスからwhereが生えているコード

```php
<?php

$user = User::find(1);

// User::where('id', 1)と同じように動く
$user->where('id', 1);
```

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

# Eloquentの抽象構造

- Eloquent\Modelを継承したクラス→モデル
  - **データベースのテーブル、行を表現**
  - **クラスとしてはテーブルを、インスタンスとしては行を**
- データベースとオブジェクトのギャップの対処
- データベースの機能を超えた部分
- 外部連携
- ユーティリティ的な機能

<!--

(4-2章)

ということで、Eloquentの全貌、なんとなく見えてきた気がします。
まとめです。
Eloquentの中心となるのは、Eloquent\Modelを継承したクラスです。
たとえばQuery\BuilderがSQLクエリと、
その実行を全面的に表現しているのに対して、
モデルクラスが表現するのは特定のテーブルと、その行です。

クラスとして使用する場合はテーブルを、
インスタンスとして使用する場合は、
さらにテーブル内の特定の行を表現していると考えてよさそうです。

-->

---

# Eloquentの抽象構造

- Eloquent\Modelを継承したクラス→モデル
- データベースとオブジェクトのギャップの対処
  - **カラムの型のマッピング。アクセサ・ミューテタ、キャスト**
  - **カラムの値のライフサイクルがデータベースとEloquentで違うことへの対処**
- データベースの機能を超えた部分
- 外部連携
- ユーティリティ的な機能

<!--

重要な機能として、データベースとオブジェクトの違いの吸収、
というのがあると思います。

データベースから取得したカラムは、型がないわけではないのですが、
データベースの型はオブジェクトの世界のものよりはプリミティブです。
データベースの世界にオブジェクトはありませんし、
少なくともEloquentの裏にある、PDO経由では日付はDateTime型に
変更されたりせず、文字列で取得されます。

その辺をいい感じにマッピングしたり、
また、データベースとEloquentでデータが同期していないことによる
ギャップを埋めるための独自のライフサイクルがあったりします。

-->

---

# Eloquentの抽象構造

- Eloquent\Modelを継承したクラス→モデル
- データベースとオブジェクトのギャップの対処
- データベースの機能を超えた部分
  - **リレーションはオブジェクトのリンクの形に**
  - **グローバルスコープという、全体への制約**
  - **ローカルスコープは実装としては小さい機能だけど……**
  - **行の直接操作。行を直接update, delete**
- 外部連携
- ユーティリティ的な機能

<!--

データベースの機能を超えた部分、と考ええられるものもあります。
リレーションは、データベースのものとは形も違いますし、
扱いやすさも違います。

グローバルスコープやローカルスコープ、
ローカルスコープは、実装は非常に小さな機能ですが、
SQLの大きな問題の一つ、WHERE句の条件を再利用できないという
問題をシンプルに解決する強力な機能です。

行を抽象化して、直接操作するupdate, deleteのようなメソッドもあります。

-->

---

# Eloquentの抽象構造

- Eloquent\Modelを継承したクラス→モデル
- データベースとオブジェクトのギャップの対処
- データベースの機能を超えた部分
- 外部連携
  - シリアライズ
  - 入出力セキュリティ
  - ブロードキャスト
  - ページネーション
- ユーティリティ的な機能

<!--

Laravel内のほかの機能や、外部との連携のための機能もいろいろあります。

-->

---

# Eloquentの抽象構造

- Eloquent\Modelを継承したクラス→モデル
- データベースとオブジェクトのギャップの対処
- データベースの機能を超えた部分
- 外部連携
- ユーティリティ的な機能
  - ブート処理
  - `is` `isNot`のようなオブジェクトの比較
  - `replicate`のようなオブジェクトの複製
  - 関連オブジェクトを生成するファクトリメソッド
  - インターフェイスを実装するためのメソッド

<!--

もちろん、高機能なクラスとしてやっていくための、
細かな、ユーティリティ的な機能もいろいろあります。
Eloquentの全体像の中では、おまけみたいなものですが。

-->

---

# Eloquentの物理構造

- Eloquent\Model
  - Eloquent\Builder
    - Query\Builder
  - HasAttributes
  - HasRelationships
    - Eloquent\Relations\Relation
  - その他トレイト

<!--

そして、これらEloquentの抽象的な構造、機能を実現する、
物理的な構造は、こんな感じです。

Eloquent\Modelがやはり中心となり、Eloquent\Builder,
Query\Builderに委譲していく方面、
残りは主にEloquent\Modelの本体やトレイトに実装されていて、
アトリビュート回りをHasAttributesが、
リレーション回りをHasRelationshipsと、Eloquent\Relations\Relationを
継承したクラスが、
それぞれ中心となって実装しています。

細かいところはもっといろいろあるわけですが、とりあえずこの辺を
抑えておけば、後は必要なときに見に行けば足りるかなと思います。

-->

---

# 私が理解したEloquentの全貌

- SQL/RDBMSの、テーブルと行をクラスとインスタンスで表現しつつ、強力なリレーション機能と、関連する細々とした機能を持つモジュール

<!--

私が理解したEloquentの全貌を一言でまとめてみます。

「SQL/RDBMSの、テーブルと行をクラスとインスタンスで表現しつつ、
強力なリレーション機能と、関連する細々とした機能を持つモジュール」

こんな感じでいかがでしょうか？　以上です。

-->

---

![bg right:33%](icon-DSC_0417.JPG)

# 自己紹介

- HAYASHI Masayuki (Twitter: @hayashi_msyk)
- 神戸在住
- フリーランスのサーバサイドプログラマ
  - ♥PHP/Laravel
  - だったんだけど、最近はなぜかRailsメインの会社でフロントエンド(Vue/TypeScript)書いてます
  - 週24時間労働実践中
* **カンファレンス初登壇！**

<!--

最後になりましたが、簡単に自己紹介です。
林と申します。神戸に住んでます。

生まれは北海道で、日本の逆の端、沖縄には強い憧れがあります。
が、今回も初沖縄は果たせませんでした……。
コロナ落ち着いたら、今度こそ行きたいですね。

フリーランスでやってます。PHP/Laravelの人だったんですが、
最近はなぜかフロントエンドやってます。
これはこれで楽しいですね。でもLaravelもまた書きたい。

(めくる)
カンファレンス初登壇でした。緊張しました。いかがでしたでしょうか。
ご意見ご感想などありましたら、ぜひご連絡ください！

-->

---

# ご清聴ありがとうございました

<style scoped>
h1 {
  text-align: center;
}
</style>

<!--

(～53分)

-->

