---
title: "夏休みの終わりこそ復習しておきたい、ES2016以降のモダンJavaScript再入門"
emoji: "🌻"
type: "tech"
topics:
  - "javascript"
published: false
publication_name: "cybozu_frontend"
---

:::message
この記事は、[CYBOZU SUMMER BLOG FES '25](https://cybozu.github.io/summer-blog-fes-2025/)の記事です。
:::

## はじめに
夏も終わりに近づいてきましたね。みなさん夏休みはいかがお過ごしでしょうか？海に山に、はたまたエアコンの効いた部屋でJavaScriptでコーディング三昧もアリかなと思ったりもします。ところで、コーヒー片手にコードを書きながら、「うわっ...私のJavaScript、古すぎない...？」とふと思ったことがありませんか？

ECMAScript 6(以下ES6)が正式リリースされた2015年からかれこれ10年も経ちましたね。ES6はPromise、クラス構文やアロー関数など強力な機能を一気に導入しました。ES3から約10年間で標準仕様の策定がほぼ停滞状態だったJavaScriptにとって起死回生と言っても過言ではない節目でした。

もし手元にTypeScriptを使っているプロジェクトがあれば、`tsconfig.json`を覗いてほしい。高い確率でコンパイルの`target`に`ES6`もしくは`ES2015`が設定されているはずです。それほど時代に大きなインパクトを残したES6ですが、実はそれを境に毎年6月頃に新しいバージョンがリリースされるようになりました。ES2016、ES2017、ES2018...と続き、現在はES2025まで策定されていて、毎年着実に進化を続けています。

ただ、JavaScriptとして素晴らしいところでもあり、呪いでもあるのが「後方互換性の維持」です。後方互換性が保たれることで安心感を得る一方、古いコードが動き続けるがゆえに現場のコードベースは「動くからいいや」と古いスタイルのままになりがちです。新しい便利な機能があるのに、それを知らずに苦労している...そんな「あるある」な状況、心当たりありませんか？

例えば：
```javascript
// 古き良き書き方(今でも動く)
var lastItem = array[array.length - 1];

// よりモダン(ES2022+)な書き方
const lastItem = array.at(-1);
```

この記事では、ES2016以降に追加された「明日から使える」便利な機能を私の主観でピックアップして紹介します。夏休みの締めくくりに、モダンJavaScriptの知識をアップデートして、秋からの開発を少し楽にしてみませんか？

## 言語機能の強化
### `globalThis`オブジェクト (ES2020)

ブラウザでは`window`、Node.jsでは`global`、Web Workerでは`self`...環境ごとにバラバラだったグローバルオブジェクトへのアクセス方法を統一したい、というエンジニアたちの願いを叶えてくれた待望の機能です：

```javascript
// Before - グローバルオブジェクトを取るために環境で分岐書しないといけなかった
const globalObj = (function() {
  if (typeof window !== 'undefined') return window;
  if (typeof global !== 'undefined') return global;
  if (typeof self !== 'undefined') return self;
  throw new Error('グローバルオブジェクトが見つかりません！');
})();

// After - 1行でどの環境でも動く！🎉
const globalObj = globalThis;
```

Universal JavaScriptを書く際に特に役に立ちます。この環境判定用のボイラープレートをコードベース内に見つけたら迷わず置き換えましょう！

### 末尾カンマ(ES1~ES2017)

末尾カンマという仕様は実はかなりの昔（ES1）から使えるようになっていたが、意外と知らない人が多いです。

簡単に言うと、配列(ES1)とオブジェクト(ES5)及び関数の引数リスト(ES2017)の最後にカンマを付けてもシンタックスエラーにならない仕様です。地味ですが、実はものすごく実用的な機能です。

```javascript
// 配列(ES1から使用可能)
var arr = [
    'es3',
    'es5',
    'es2025', // ← この末尾カンマ
  ];

// オブジェクト(ES5から使用可能)
  const member = {
    es1: 'array',
    es5: 'object',
    es2017: 'function', // ← 末尾カンマ
  };

// 関数定義(ES2017から使用可能)
function findProposal(
  language,
  year,
  stage,  // ← 末尾カンマ
) {
  // ...
}

// 呼び出し時も同様
findProposal(
  'JavaScript',
  '2025',
  3,  // ← これもOK
);
```

「たかがカンマ、されどカンマ」ですが、末尾の項目の位置を変更する時にいちいちカンマの存在を気にしなくて済むので、個人的にオススメのスタイルです。

また、項目を追加・削除した際にGitのdiffがスッキリになるという恩恵も受けられます.

```diff
// 末尾カンマ使わない場合の差分
 createUser(
   'Taro',
   'taro@example.com',
-  25
+  25,
+  'Tokyo'
 );

// 末尾カンマ使う場合の差分
 createUser(
   'Taro',
   'taro@example.com',
   25,
+  'Tokyo',
 );
```

ただし、Prettierなどのフォーマッターが大体気が利いて末尾のカンマを除去してくれる挙動がデフォルトになっているので、`trailingComma: "all"`と設定しておくと良いでしょう！

### Optional Catch Binding(ES2019)

`try...catch`構文で、catch節の引数（エラーオブジェクト）を省略できる仕様がES2019で導入されました。

```javascript
// Before - エラーオブジェクトは取ったけど...
try {
  doSomethingRisky();
} catch (e) {  // 使わないのになぁ...
  console.log('エラーが発生しました');
  rollback();
}

// After- 使わなければcatch節の引数を省略
try {
  doSomethingRisky();
} catch {  // ｽｯｷﾘ✨
  console.log('エラーが発生しました');
  rollback();
}
```

「エラーの詳細はどうでもいいから、自前で処理を書きたい」といった場面においては便利です。が、濫用するとエラーを握りつぶすことにつながりやすい面もあるので、適切にエラーハンドリングができていると判断した時のみ使いましょう。

### オブジェクトに使うRest & Spread構文 (ES2018)

E6から配列に使えていた便利なRest / Spread構文(`...`)ですが、ES2018でオブジェクトにも使用可能となりました。Reactで開発している人は四六時中使っているはずです。

```javascript
// Spread = オブジェクトの展開
const user = { name: 'Taro', age: 25 };
const updatedUser = { ...user, age: 26 };  // シャローコピー & 更新

// Rest = プロパティをまとめる
const { password, ...publicUserData } = userData;
// publicUserData = passwordを除外したオブジェクト（安全！）

// オブジェクトの結合(マージ)
const defaults = { theme: 'light', fontSize: 14 };
const userPrefs = { fontSize: 16 };
const finalSettings = { ...defaults, ...userPrefs };  // fontSizeが16になる
```

構文としての見た目が同じ`...`なので混同しやすいですが、**「集めるのがRest、広げるのがSpread」** と覚えましょう。つまり、式の左辺にあればRest、右辺にあればSpreadです。

:::message alert
Spread構文はシャローコピーとなるで、ネストされたオブジェクトは参照が共有されます。ディープコピーが必要な場合は`structuredClone()`を使いましょう。
:::

### Null合体演算子(`??`) (ES2020)

`??`の左辺が`null`または`undefined`の場合(=nullish)にのみ、右辺の値を返す演算子です。

論理和の`||`演算子とよく混同されやすいですが、`||`演算子falsyな値で判定するのに対し、`??`はnullishな値のみで判定する点が使い分けのポイントです。

```javascript
// ||演算子との違い
const count1 = 0 || 10;        // count1 = 10（0はfalsyなので）
const count2 = 0 ?? 10;        // count1 = 0（0はnullishではない）

const name1 = '' || 'デフォルト';   // name1 = 'デフォルト'（空文字はfalsy）
const name2 = '' ?? 'デフォルト';   // name1 = ''（空文字はnullishではない）
```

このように、昔`||`演算子使用しか使えなかった時代でよく悩まされていた「0や空文字が意図せずデフォルト値になってしまう問題」をスパッと解決してくれます。

次に出てくるOptional Chaining (`?.`)と非常に相性が良く、「安全にアクセスして、値がなければデフォルト値を使う」という場面ではセットで使われることが多いです。

### Optional Chaining(`?.`)(ES2020)

ネストされたオブジェクトの`null`や`undefined`なプロパティにアクセスする際に、JavaScriptの開発者なら誰でも一度ぐらい遭遇したことがある例の忌々しい`Cannot read properties of undefined (reading 'プロパティ名')`エラーの発生を抑止し、代わりに`undefined`を返してくれる、プロパティに安全にアクセスするための構文です。

```javascript
// Before - Optional Chaining使わないと...
const users = [
    { name: 'John', address: { city: 'New York' } },
    { name: 'Mary' },
    { name: 'Sophie', address: { country: 'Singapore'} }
];
for (let user of users) {
    // ❌ この辺でCannot read properties of undefinedとなる
    const city = user.address.city.toUpperCase();
    console.log(city ?? `No city defined for ${user.name}`);
}

// After - エラーにならずに出力してくれる
const users = [
    { name: 'John', address: { city: 'New York' } },
    { name: 'Mary' },
    { name: 'Sophie', address: { country: 'Singapore'} }
];
for (let user of users) {
    // ✅ addressからundefinedでもNo Cityと出力してくれる
    const city = user.address?.city?.toUpperCase();
    // Null合体演算子と合体して使う
    console.log(city ?? `No city defined for ${user.name}`);
}

// 実はメソッドの呼び出しにも使える
const result = obj.method?.();  // methodが存在する場合のみ実行

// 配列へのアクセスにも使える
const firstItem = array?.[0];
```

便利な構文ですが、何もかも「とりあえず`?`を付けておけば安全だろう」という考え方はプロジェクトのメンテナンス性を低下させるので、どこが`nullish`になり得るかをきちんと把握してから使いましょう。

```javascript
// ❌ 悪い例：全部に?.を付ける
const value = obj?.prop1?.prop2?.prop3?.prop4;

// ✅ 良い例：nullishになり得る箇所だけに付ける
const value = apiResponse?.data.items[0].name;
```

## 進化したクラス構文
### プライベートなクラスプロパティ (`#`) (ES2022)

クラスのプロパティやメソッドを外部から完全に隠蔽するための公式な構文です。`#`という表現に最初は戸惑う人が多いかもしれませんが、慣れると便利です。

```javascript
class BankAccount {
  #balance = 0;  // プライベートフィールド
  #pin;         // 宣言だけでもOK
  
  constructor(initialBalance, pin) {
    this.#balance = initialBalance;
    this.#pin = pin;
  }
  
  // プライベートメソッド
  #validatePin(inputPin) {
    return this.#pin === inputPin;
  }
  
  withdraw(amount, pin) {
    if (!this.#validatePin(pin)) {
      throw new Error('PINが間違っています');
    }
    if (this.#balance < amount) {
      throw new Error('残高不足です');
    }
    this.#balance -= amount;
    return this.#balance;
  }
  
  get balance() {
    return this.#balance; // getter経由でのみアクセス可能
  }
}

const account = new BankAccount(10000, '1234');
console.log(account.balance); // 10000
// console.log(account.#balance); // SyntaxError! 外部からアクセス不可
```
TypeScriptを使い慣れている人から見ると「private修飾子と同じだろう？」と思われがちですが、実はランタイムで大きな違いが出てきます。

```typescript
// TypeScriptのprivate（コンパイル後は消えてただのプロパティとなる）
class TypeScriptClass {
  private secret = 'ランタイムでは普通のプロパティ';
}

// JavaScriptの#（ランタイムでもプライベート）
class JavaScriptClass {
  #secret = '絶対に外部からアクセスできない';
}
```

このように、TypeScriptのprivate修飾子はあくまでコンパイル時のチェックに過ぎず、ランタイムでは普通のプロパティになります。一方、`#`を使ったプライベートフィールドはランタイムでも完全にプライベートです。セキュリティが重要な場面では#の使用を検討しましょう。

### staticフィールド & staticブロック(ES2022)

staticフィールドと、クラスの初期化処理を記述できるstaticブロックがES2022で追加されました。`new`キーワードでインスタンスを作らなくても、クラス自体に紐づく値や処理を定義できます。

```javascript
class Config {
  // staticフィールド
  static VERSION = '2.0.0';
  static API_ENDPOINT = 'https://api.example.com';
  static #privateConfig = new Map();
  
  // staticブロック(クラスの初期化時に1度だけ実行)
  static {
    // 環境に応じた初期設定
    if (process.env.NODE_ENV === 'development') {
      this.API_ENDPOINT = 'http://localhost:3000';
    }
    
    // プライベートな設定の初期化
    this.#privateConfig.set('timeout', 5000);
    this.#privateConfig.set('retries', 3);
    
    console.log(`Config initialized for ${this.VERSION}`);
  }
  
  static getConfig(key) {
    return this.#privateConfig.get(key);
  }
}

console.log(Config.VERSION);        // '2.0.0'
console.log(Config.getConfig('timeout')); // 5000
```

staticブロックは、複雑な初期化ロジックをクラス定義の中で完結させられるところが利点です。グローバルスコープを汚さずに、クラスに関連する初期化処理を整理する時にぜひご活用ください。

## 便利なメソッド - オブジェクト操作編

ES6以降、オブジェクトを操作するための便利なメソッドが続々と追加されています。リリース順に見ていきましょう。

### `Object.values()` / `Object.entries()`(ES2017)

これまで`Object.keys()`でキーを取得してから値にアクセスしていた処理が、格段にシンプルになりました。特に`Object.entries()`は、オブジェクトを配列のメソッド（map、filter、reduceなど）で処理したい時に威力を発揮します。「オブジェクトなのに`for...of`感覚で回せる！」という感動を味わった人が少なくないはずです。

```javascript
const scores = { math: 90, english: 85, science: 92 };

// values() - 値だけ取り出す
const values = Object.values(scores);
console.log(values); // [90, 85, 92]
console.log(`平均点: ${values.reduce((a, b) => a + b) / values.length}`);

// entries() - キーと値のペアを配列で取得
const entries = Object.entries(scores);
console.log(entries); // [['math', 90], ['english', 85], ['science', 92]]

// entries() - 次項のfromEntries()との組み合わせでオブジェクトの変換処理がより簡単に
const doubledScores = Object.fromEntries(
  Object.entries(scores).map(([subject, score]) => [subject, score * 2])
);
console.log(doubledScores); // { math: 180, english: 170, science: 184 }
```

### `Object.fromEntries()`(ES2019)

`Object.entries()`で配列化したオブジェクトを、再びオブジェクトに戻すための逆操作を行うメソッドです。配列のメソッドでフィルタリングや変換を行った後、最後にオブジェクトに戻すというパイプライン処理が美しく書けるようになりました。また、`Map`や`URLSearchParams`をオブジェクトに変換する際にも重宝します。まさに「行って帰ってこれる」安心感のあるメソッドです。

```javascript
// Object.entries()の逆操作
const entries = [['name', 'Taro'], ['age', 25]];
const user = Object.fromEntries(entries);  // { name: 'Taro', age: 25 }

// MapをObjectに変換する場面でもよく使われる
const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
const obj = Object.fromEntries(map); // { key1: 'value1', key2: 'value2' }
```

### `Object.hasOwn()`(ES2022)

昔、オブジェクト自身が特定のプロパティを所持しているかどうかを判別するために`hasOwnProperty()`使っていました。しかしながら、`hasOwnProperty()`はプロトタイプチェーンを通じて継承されるメソッドであるため、少なくとも以下の2つの問題がありました：

1. **オーバーライドされる危険性**: オブジェクト自身に`hasOwnProperty`プロパティが定義されていると、本来のメソッドが隠蔽されてしまう。
2. **null安全ではない**: 例えば`Object.create(null)`で作成したオブジェクトは`hasOwnProperty`メソッドを持たないので、呼び出そうとするとエラーで落ちてしまう。

```javascript
const maliciousObj = {
  hasOwnProperty: function() {
    return true;  // 常にtrueを返す悪意のある実装
  }
};
maliciousObj.hasOwnProperty('anyProp');  // true（嘘の結果）

const nullObj = Object.create(null);
nullObj.hasOwnProperty('prop');  // TypeError! hasOwnPropertyが存在しない
```

よって、従来だとわざわざObjectから長いメソッドのコールでそれらの問題点を回避していました：

```javascript
// 従来の方法（長い...）
if (Object.prototype.hasOwnProperty.call(person, 'name')) {
  console.log('nameプロパティあり');
}
```

`Object.hasOwn()`はまさにこの問題に対処するためにリリースされたObject直属の静的メソッドでした：

```javascript
// より安全！
Object.hasOwn(obj, 'prop');

// オーバーライドも怖くない！
const maliciousObj = {
  hasOwnProperty: function() {
    return true;  // 常にtrueを返す悪意のある実装
  }
};
Object.hasOwn(maliciousObj, 'anyProp');  // false（正しい結果）

// nullオブジェクトでもOK！
const nullObj = Object.create(null);
Object.hasOwn(nullObj, 'prop');     // false（エラーにならない）
```

これからセキュリティやエッジケースを考慮したコードを書く際は、もう迷わず`Object.hasOwn()`を選びましょう！

## 便利なメソッド - 文字列操作編

文字列を便利に操作するためのメソッドもES6以降に続々と追加されました。サンプルコードで一気に見ていきましょう。

```javascript
// 📏 padStart() / padEnd() (ES2017) - 埋め込む
const id = '42';
console.log(id.padStart(5, '0'));  // '00042' (ゼロパディング)
console.log('Loading'.padEnd(10, '.')); // 'Loading...'

// 時刻表示によく使う
const hours = '9';
const minutes = '5';
const time = `${hours.padStart(2, '0')}:${minutes.padStart(2, '0')}`;
console.log(time); // '09:05'

// ✂️ trimStart() / trimEnd() (ES2019) - 空白文字を除去
const message = '  Hello World  ';
console.log(message.trimStart()); // 'Hello World  '
console.log(message.trimEnd());   // '  Hello World'

// 🔍 matchAll() (ES2020) - 正規表現に当てはまった文字列を配列で取得
const text = `
    The project started on 2021-03-15 and the first phase was completed by 2021-04-20.
    We had a review meeting on 2021-05-10. The final deadline was 2021-06-30, which we met successfully.
`;

const dateRegex = /\b(\d{4})-(\d{2})-(\d{2})\b/g;
const matches = text.matchAll(dateRegex);

for (const match of matches) {
    console.log(`Found date: ${match[0]}`);
    console.log(`Year: ${match[1]}, Month: ${match[2]}, Day: ${match[3]}`);
}
/* 出力：
 * Found date: 2021-03-15
 * Year: 2021, Month: 03, Day: 15
 * Found date: 2021-04-20
 * Year: 2021, Month: 04, Day: 20
 * Found date: 2021-05-10
 * Year: 2021, Month: 05, Day: 10
 * Found date: 2021-06-30
 * Year: 2021, Month: 06, Day: 30
 */

// 🎯 replaceAll() (ES2021) - まとめて置換、ついに来た！
const template = 'Hello {name}, welcome to {place}!';

// Before - 正規表現のgフラグが必要だった
const old = template.replace(/{name}/g, '太郎').replace(/{place}/g, '東京');

// After - 直感的に書ける！
const text2 = template.replaceAll('{name}', '太郎').replaceAll('{place}', '東京');
console.log(text2); // 'Hello 太郎, welcome to 東京!'
```

ちなみに、文字列操作に関しては、同じく[CYBOZU SUMMER BLOG FES '25](https://cybozu.github.io/summer-blog-fes-2025/)に参加している同僚のおぐえもんさんによる、素晴らしい記事も公開されています。こちらもぜひご一読ください🙌
https://zenn.dev/cybozu_frontend/articles/js_basic_string

## 便利なメソッド - 配列操作編

次に、配列操作をより宣言的で、そして安全に取り扱うためのメソッドたちを見ていきましょう。

### `Array.prototype.includes()` (ES2016)

配列に特定の要素が含まれているかを判定する、より直感的なメソッドです。

```javascript
const numbers = [1, 2, 3, NaN];

// Before - indexOfの問題点
console.log(numbers.indexOf(NaN)); // -1 (NaNを見つけられない！)
console.log(numbers.indexOf(2) !== -1); // true (読みづらい...)

// After - includesなら直感的！
console.log(numbers.includes(NaN)); // true (NaNも正しく判定)
console.log(numbers.includes(2));   // true (シンプル！)

// 例：権限チェック
const userRoles = ['user', 'editor'];
if (userRoles.includes('admin')) {
  console.log('管理者権限あり');
}

// 第2引数で検索開始位置も指定可能
const fruits = ['apple', 'banana', 'apple', 'orange'];
console.log(fruits.includes('apple', 2)); // true (3番目以降から検索)
```

耳寄りな情報ですが、`includes()`は`===`による厳密等価ではなく、SameValueZeroという比較アルゴリズムを使用しているため、これにより`[NaN].includes(NaN)`を`true`で判定できています。

### `Array.prototype.flat() / flatMap()` (ES2019)

ネストされた配列をフラット化する待望のメソッドです。

```javascript
// flat() - 配列をフラット化。引数で階層の深さを指定できます
const nested = [1, [2, 3], [4, [5, 6]]];
console.log(nested.flat());    // [1, 2, 3, 4, [5, 6]] (1階層だけ)
console.log(nested.flat(2));   // [1, 2, 3, 4, 5, 6] (2階層)
console.log(nested.flat(Infinity)); // 全階層を平坦化にInfinityを使いましょう！

// 実は空要素を除去してくれる
const sparse = [1, , , 2, , 3];
console.log(sparse.flat()); // [1, 2, 3]

// flatMap() - mapしてからflatを行う（1階層のみ）
const sentences = ['Hello World', 'Good Morning'];

// Before - mapしてからflat
const words1 = sentences.map(s => s.split(' ')).flat();

// After - flatMapで完結！
const words2 = sentences.flatMap(s => s.split(' '));
console.log(words2); // ['Hello', 'World', 'Good', 'Morning']

// 実例：複数のAPIレスポンスの統合
const apiResponses = [
  { data: [1, 2, 3] },
  { data: [4, 5] },
  { data: [6, 7, 8] }
];
const allData = apiResponses.flatMap(response => response.data);
console.log(allData); // [1, 2, 3, 4, 5, 6, 7, 8]

// 条件によって要素を増減させることも可能
const numbers2 = [1, 2, 3, 4, 5];
const doubled = numbers2.flatMap(n => 
  n % 2 === 0 ? [n, n] : n  // 偶数は2つに、奇数はそのまま
);
console.log(doubled); // [1, 2, 2, 3, 4, 4, 5]
```

`flatMap()`という名前から「flatが先じゃないのか？」と誤解されがちですが、実際は「mapしてからflat(1)する」という順序です。Java経験者なら`Stream.flatMap()`でお馴染みの概念ですね。

なお、`flatMap()`は常に1階層までしかフラット化できないので、より深い階層の配列には`map().flat(n)`を使いましょう。

### `Array.prototype.at()`(ES2022)

配列の要素を直接にインデックスで指定して取れるようになった待望のメソッドがついにES2022で実装されました。なんと負のインデックスも使えて末尾から要素を探せる優れものです。Pythonユーザーもさぞかし羨ましいでしょう。

```javascript
const colors = ['red', 'green', 'blue', 'yellow'];

// Before - 末尾の要素を取得するのが面倒
const lastOld = colors[colors.length - 1]; // 'yellow'
const secondLastOld = colors[colors.length - 2]; // 'blue'

// After - at()でスッキリ！
const last = colors.at(-1);  // 'yellow'
const secondLast = colors.at(-2); // 'blue'
const first = colors.at(0);  // 'red' (正のインデックスも使える)

// 実例：パスの最後の部分を取得
const path = '/users/profile/settings'.split('/');
const currentPage = path.at(-1); // 'settings'

// ファイルの拡張子の取得でも重宝
function getExtension(filename) {
  return filename.split('.').at(-1);
}
console.log(getExtension('document.pdf')); // 'pdf'
console.log(getExtension('archive.tar.gz')); // 'gz'
```

当たり前のような話ですが、`at()`文字列でも使えます。これでようやく`str[str.length - 1]`みたいな冗長な書き方から解放されます：

```javascript
const message = 'Hello!';
console.log(message.at(-1)); // '!'
```

### `Array.prototype.to[Verb]ed()` & `with()`(ES2023)

`sort()`や`splice()`のような元の配列に破壊的な変更を加えるメソッドに対し、新しい配列を返す非破壊的なバージョンが遂に登場しました。Reactなどイミュータブルな操作が推奨されるフレームワークでの開発がより快適になりました。

```javascript
// toReversed()
const numbers = [1, 2, 3];
const reversedNumbers = numbers.toReversed();
console.log(reversedNumbers); // [3, 2, 1](新しい配列)
console.log(numbers);         // [1, 2, 3](元のまま)

// toSorted()
const unsortedNumbers = [3, 1, 2];
const sortedNumbers = unsortedNumbers.toSorted();
console.log(sortedNumbers);   // [1, 2, 3](新しい配列)
console.log(unsortedNumbers); // [3, 1, 2](元のまま)

// toSpliced()
const originalArray = [1, 2, 3, 4, 5];
const splicedArray = originalArray.toSpliced(1, 2, 'a', 'b');
console.log(splicedArray);  // [1, a, b, 4, 5](新しい配列)
console.log(originalArray); // [1, 2, 3, 4, 5](元のまま)
```

また、少しだけ毛色が違うが、文字列の`replace()`と同等な非破壊的変更を配列でも可能にする`with()`という新たなメソッドも追加されました。

```javascript
const items = ['pencil', 'pen', 'eraser'];
const updated = items.with(1, 'marker');
console.log(updated); // ['pencil', 'marker', 'eraser'](新しい配列)
console.log(items);   // ['pencil', 'pen', 'eraser'](元のまま)

// -1で末尾の要素を置き換える操作もより簡単に！
const lastUpdated = items.with(-1, 'ruler');
console.log(lastUpdated); // ['pencil', 'pen', 'ruler']
```

これらのイミュータブルな操作を行うメソッドの登場によって、Reactの状態更新などでいつもSpread構文に頼っていた部分がすっきりと記述できるようになりました。

```javascript
// Before - スプレッド構文で頑張る
const [todos, setTodos] = useState([...]);
setTodos([...todos].sort((a, b) => a.priority - b.priority));

// After - より直感的な書き味(可読性もUP!)
setTodos(todos.toSorted((a, b) => a.priority - b.priority));

// インデックス指定の状態更新もより簡単に
setTodos(todos.with(index, { ...todos[index], completed: true }));
```

## 非同期処理をより簡潔に - async/await(ES2017)

「あれ、このふたりってES6からじゃないの？」と思われがちな`async/await`構文ですが、実はES2017からです。ES6で導入された`Promise`をより直感的に書けるようにしたシンタックスシュガーです。

```javascript
// Promiseチェーンで非同期処理を書く
function fetchUserData(userId) {
  return fetch(`/api/users/${userId}`)
    .then(response => response.json())
    .then(user => {
      return fetch(`/api/posts?userId=${user.id}`);
    })
    .then(response => response.json())
    .then(posts => {
      console.log('投稿を取得しました:', posts);
      return posts;
    })
    .catch(error => {
      console.error('エラー:', error);
    });
}

// async/awaitを使った書き方
async function fetchUserData(userId) {
  try {
    const userResponse = await fetch(`/api/users/${userId}`);
    const user = await userResponse.json();
    
    const postsResponse = await fetch(`/api/posts?userId=${user.id}`);
    const posts = await postsResponse.json();
    
    console.log('投稿を取得しました:', posts);
    return posts;
  } catch (error) {
    console.error('エラー:', error);
  }
}
```

ここでよくある誤解として、`async/await`が非同期処理に何か革新的な仕組みをもたらした、というものがありますが、実際はそうではありません。`async/await`はあくまでPromiseの**書き方を変えただけ**で、非同期処理の仕組み自体は、依然としてPromiseベースですーーあくまで**シンタックスシュガー**ですからね。

逆に、パフォーマンスが重要な場面では、あえて`Promise.all`を活用した方が良いケースもよくあります。

```javascript
// ❌ 順番に実行（遅い）
const user = await fetchUser();
const posts = await fetchPosts();
const comments = await fetchComments();

// ✅ 並列実行（速い）
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);
```

## ES2024 & ES2025に入った新機能ピックアップ

ここまで紹介してきたES2017〜2023の機能の多くは、既に現場で活用されているでしょう。しかし、JavaScriptの進化はまだまだ止まっていません。

ES2024は去年の6月に正式リリースされ、ES2025も今年の6月に正式リリースされました。これらのリリースによって導入された新機能は、各ブラウザが既にサポートし始めているものの、開発現場にはまだそれほど浸透していないはずです。こちらもまとめて見ておきましょう。

### `[Object/Map].groupBy()`(ES2024)

これまでLodashの`_.groupBy()`に頼っていた配列の要素をグループ化する処理ですが、遂に標準仕様となりました。

```javascript
const users = [
  { name: 'Alice', age: 25, role: 'admin' },
  { name: 'Bob', age: 30, role: 'user' },
  { name: 'Charlie', age: 25, role: 'user' },
  { name: 'David', age: 30, role: 'admin' }
];

// Object.groupBy()
const byAge = Object.groupBy(users, user => user.age);
/* 
 * byAge = 年齢でグルーピングされたユーザー情報(Object)
 * {
 *   25: [{ name: 'Alice', ... }, { name: 'Charlie', ... }],
 *   30: [{ name: 'Bob', ... }, { name: 'David', ... }]
 * }
 */

// Map.groupBy()
const byRole = Map.groupBy(users, user => user.role);
/*
 * byRole = ロールでグルーピングされたユーザー情報(Mapインスタンス)
 * Map(2) {
 *   'admin' => [{ name: 'Alice', ... }, { name: 'David', ... }],
 *   'user' => [{ name: 'Bob', ... }, { name: 'Charlie', ... }]
 * }
 *  
*/

// 複雑な条件を指定してグルーピングも可能
const byAgeRange = Object.groupBy(users, user => {
  if (user.age < 30) return 'young';
  if (user.age < 40) return 'middle';
  return 'senior';
});

/* 
 * byAgeRange
 * {
 *    young: [
 *      { name: 'Alice', age: 25, role: 'admin' },
 *      { name: 'Charlie', age: 25, role: 'user' }
 *    ],
 *    middle: [
 *      { name: 'Bob', age: 30, role: 'user' },
 *      { name: 'David', age: 30, role: 'admin' }
 *    ]
 * }
*/
```

`Object`と`Map`の使い分けは、**キーの型で**決める良いでしょう。文字列や数値をキーにしたいなら`Object.groupBy()`を使用し、オブジェクトなどをキーにしたい場合は`Map.groupBy()`が適しています。

### `Promise.withResolvers()`(ES2024)

Promiseの`resolve`と`reject`関数を、Promiseオブジェクト本体と一緒に取得できる便利なメソッドです。これまでコンストラクタのコールバック内でしか使えなかった`resolve/reject`ですが、外側のスコープでも自由に使えるようになりました。

```javascript
// Before - コンストラクタの中でresolve/rejectを取る
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});

// After - 分割代入で一発で取得
const { promise, resolve, reject } = Promise.withResolvers();

// 実例：Node.jsでよく実装するタイムアウト付きのイベント待機関数
function waitForEvent(emitter, eventName, timeout = 5000) {
  const { promise, resolve, reject } = Promise.withResolvers();
  
  const timer = setTimeout(() => {
    reject(new Error('タイムアウト'));
  }, timeout);
  
  emitter.once(eventName, (data) => {
    clearTimeout(timer);
    resolve(data);
  });
  
  return promise;
}
```

DeferredパターンやPromiseのラッパーを作る際に非常に便利です。これでコールバック地獄を避けつつ、Promiseの制御をより柔軟に行えるようになりました。

### `Set.prototype.method()`の充実化(ES2025)

`groupBy()`に続き、これまで自前で実装したり、Lodashに頼っていたSetオブジェクトでの集合演算が標準機能になりました。データ分析や重複を取り除く処理が格段にシンプルになります。

```javascript
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

// 和集合演算(union)
setA.union(setB);        // Set {1, 2, 3, 4}
// 積集合演算(intersection)
setA.intersection(setB); // Set {2, 3}
// 差集合演算(difference)
setA.difference(setB);   // Set {1}
// 公式では「対称差分」と記載しているが、いわゆるXOR演算です
setA.symmetricDifference(setB); // Set {1, 4}

// 部分集合としての判定も
setA.isSubsetOf(setB);   // false
setA.isSupersetOf(setB); // false
setA.isDisjointFrom(setB); // false（共通要素がある）
```

### Iteratorのヘルパーメソッド(ES2025)

これまでイテレータを`map`や`filter`で処理する際には、一度配列に変換する必要がありましたが、ES2025からイテレータのままで処理できるようになりました。

```javascript
// イテレータに対して直接メソッドチェーンが可能に
const numbers = [1, 2, 3, 4, 5].values()
  .filter(n => n % 2 === 0) // そのまま続けて書ける！
  .map(n => n * 2)
  .take(2) // 必要な要素まで取れたら処理を打ち切るためパフォーマンス向上にも繋がる
  .toArray();  // [4, 8]

// 無限イテレータだって取り扱える
function* infiniteNumbers() {
  let i = 0;
  while (true) yield i++;
}

const firstTenEvens = infiniteNumbers()
  .filter(n => n % 2 === 0)
  .take(10)
  .toArray();  // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

配列への変換が不要になるだけでも便利ですが、それ以上に大きなメリットがGeneratorとセットで使う場面での**遅延評価**です。配列の場合は全要素を処理してから次のメソッドに渡しますが、イテレータなら必要な分だけ処理するため、メモリ効率が格段に向上します。

## まとめ

ちょいと駆け足でES2017以降の新機能を見てきましたが、いかがでしたでしょうか？

今回は、ECMAScriptの仕様策定プロセスで **「Stage 4 (Finished)」** になった安定機能のみピックアップして紹介しました。実は他にも、`Temporal` API（日付処理にヒカリを！）のように、現在Stage 3ですがかなり注目されている目玉機能もたくさんあります。これらは今後の楽しみとして取っておきましょう！

:::message
ちなみにTC39のProposalは[こちら](https://www.proposals.es/)で確認できます。
:::

また、ここで敢えて大切なことをお伝えします：**新しい機能が常に最適解とは限りません**。

例えば、先ほど触れたように、パフォーマンスが重要な場面では`Promise.all`と`.then()`チェーンの方が`async/await`よりも優れている場合があります。また、シンプルな配列操作なら、新しいメソッドよりも古典的な`for`ループの方が読みやすいこともあるでしょう。

「古き良き書き方」と「モダンな書き方」両方を引き出しに持ち、状況に応じて最適なものを選択できるようにしておくと良いでしょう。

## 最後に 

JavaScriptは「Good Parts」と「Bad Parts」が混在する言語だと長らく言われてきました。しかし、これまでの進化ぶりを見ていると、着実に「Good Parts」が増え、開発者体験が日々向上していることを実感できるのではないでしょうか。

夏休みも残りわずか。この記事で得た知識を武器に、明日からのコーディングをもっと楽しくしていきましょう！