# 目標管理（Effective Java） 

* 本資料は目標管理の自己啓発で学習した内容まとめた資料である。
* Effective Javaについてまとめる。
参考書：Effective Java 第3版（著：ジョシュア・ブロック）
<br>

## 1. オブジェクトの生成と消滅
### 項目1. コンストラクタの代わりにstaticファクトリーメソッドを検討する
staticファクトリーメソッドとは、コンストラクタの代わりに使われるstaticメソッドのこと<br>

#### 1. メリット
* 名前をつけることができる<br>
例①<br>
確率的素数（probable prime）のBigInttegerを返すコンストラクタであるBigIntteger (int, int, Random) <br>
⇒BigIntteger.probablePrimeという名前のstaticファクトリメソッドにする<br>
<br>
例②<br>
クラスが同じシグニチャの複数のコンストラクタを必要とする場合、それらのコンストラクタをstaticファクトリメソッドに置き換え、名前を明確にすることで可読性が上がる。<br>
<br>
* コンストラクタと異なり、呼び出しごとに新たなオブジェクトを生成する必要がない<br>
⇒不変クラスは、あらかじめ生成されたインスタンスを使ったり、オブジェクトが生成されたときにインスタンスをキャッシュしたりして、それらのインスタンスを繰り返し使うことで不必要に重複したオブジェクトの生成を避ける。
<br>
例①<br>
Boolean.valueO(boolean)メソッド<br>
⇒オブジェクトを一つも生成しない<br>
<br>
例②<br>
staticファクトリメソッドは何度呼び出されても同じオブジェクトを返せる。<br>
＝どの時点でもインスタンスが存在するか制御することが可能<br>
⇒「インスタンス制御されている」という。<br>
<br>
* コンストラクタと異なり、メソッドの戻り値型の任意のサブタイプのオブジェクトを返せる<br>
⇒戻り値のクラスをpublicに定義しなくてもそのオブジェクトを返せる<br>

* 返されるオブジェクトのクラスは、入力パラメータの値に応じて呼び出しごとに変えられる<br>

#### 2. デメリット
* public、protectedのコンストラクタを持たないクラスのサブクラスはつくれない<br>
⇒privateなコンスタクタを持つクラスのサブクラスをつくれない。<br>

* プログラマがstaticファクトリメソッドを見つけることが難しい<br>
⇒クラスやインタフェースのドキュメントに注意を払ったり、共通する命名規則を順守することで問題回避する。<br>

#### 3. まとめ
たいていの場合はstaticファクトリメソッドを使用することが望ましい。publicのコンストラクタは状況に応じて使用するイメージでよい。<br>
<br>

### 項目2. 多くのコンストラクタパラメータに直面した時にはビルダーを検討する

参考：https://www.thekingsmuseum.info/entry/2015/08/04/194326

#### 1. テレスコーピング・コンストラクタ・パターン
順序に依存したパラメータを受けるコンストラクタをテレスコーピングコンストラクタと呼ぶ<br>
⇒必須フィールドのコンストラクタにオプションフィールドのコンストラクタを1つずつ追加する<br>
    * 本パターンは機能するが、多くのパラメータがある場合は読みづらい、かつ、書きづらい<br>

#### 2. JavaBeansパターン
パラメータなしのコンストラクタで初期化後、セッターでフィールドに値をセットする<br>
* 生成が複数の呼び出しに分割されるため、不整合な値になる可能性がある
* finalフィールドにできない<br>

#### 3. ビルダーパターン
テレスコーピング・コンストラクタ・パターンの安全性とJavaBeansパターンの可読性を組み合わせた方法<br>
⇒build()で目的のクラスのインスタンスを生成する<br>
* 目的のクラスのフィールドすべてにfinal属性を付与できる
* 読みやすい

#### 4. まとめ
コンストラクタやstaticファクトリメソッドが多くのパラメータを持つクラスを設計する際には、ビルダーパターンを使用するとよい<br>
<br>

### 項目3. privateのコンストラクタかenum型でシングルトン特性を強制する
シングルトンとは、そのクラスのインスタンスが1つであること。<br>

#### 1. public finalのフィールドによるシングルトン
> public class Elvis {<br>
　　public static final Elvis INSTANCE = new Elvis();<br>
　　private Elvis() {･･･}<br>
<br>
　　public void leaveTheBuilding() {･･･}<br>
}<br>

* コンストラクタをfinalにすることで、オブジェクトの生成を1回に強制する<br>

#### 2. staticファクトリメソッドによるシングルトン
> public class Elvis {<br>
　　purivate static final Elvis INSTANCE = new Elvis();<br>
　　private Elvis() {･･･}<br>
　　public static Elvis getInstance() { return INSTANCE; }<br>
<br>
　　public void leaveTheBuilding() {･･･}<br>
}<br>

* インスタンスはpurivate static finalフィールドで一度のみ初期化
* ファクトリメソッドで参照するINSTANCEは常に同一インスタンス<br>

#### 3. 単一要素を持つenum型
> public class Elvis {<br>
　　INSTANCE;<br>
　　<br>
　　public void leaveTheBuilding() {･･･}<br>
}<br>

* publicのフィールドによる方法と同じ
* 複数のインスタンス生成を阻止することができる

#### 4. まとめ
シングルトンはテストしづらいため、なるべく使用しない。
シングルトンを使用する場合は、単一要素を持つenum型を使用することが最善。<br>
<br>

### 項目4. privateのコンストラクタでインスタンス化不可能を強制する 
>　// インスタンス化できないユーティリティクラス<br>
public class UtiltyClass {<br>
　　<br>
　　// インスタンス化できないように、デフォルトコンストラクタを抑制<br>
　　private UtiltyClass () {}<br>
　　　　throw new AssertionError();<br>
　　}<br>
　　･･･<br>
}<br>
* サブクラス化は不可能<br>
* デフォルトコンストラクタを抑制<br>
<br>

### 項目5. 資源を直接結びつけるよりも依存性注入を選ぶ
#### 1. 静的なユーティリティの不適切な使用 － 柔軟性に欠ける。テストできない
>　public class SpellChecker {<br>
　　private static final Lexicon dictionary = ･･･;<br>
<br>
　　private SpellChecker(･･･) {}　// インスタンス化できない<br>
<br>
　　public static boolean isValid(String word) {･･･}<br>
　　public static List＜String＞ suggestions(String typo) {･･･}<br>
}<br>

#### 2. シングルトンの不適切な使用 － 柔軟性に欠ける。テストできない
>　public class SpellChecker {<br>
　　private final Lexicon dictionary = ･･･;<br>
<br>
　　private SpellChecker(･･･) {}<br>
　　public static SpellChecker INSTTANCE = new SpellChecker(･･･);<br>
<br>
　　public static boolean isValid(String word) {･･･}<br>
　　public static List＜String＞ suggestions(String typo) {･･･}<br>
}<br>

#### 3. 依存性注入 － 柔軟でありテストできる
上記、いずれの方法（静的なユーティリティ、シングルトン）も使う辞書が一つしかないと想定しているためよくない。<br>
必要なのは、新しいインスタンスを生成するときにコンストラクタに資源を渡すこと。
>　public class SpellChecker {<br>
　　private final Lexicon dictionary;<br>
<br>
　　public SpellChecker(Lexicon dictionary) {<br>
　　　　this.dictionary = Obects.requireNonNull(dictionary);<br>
　　}<br>
<br>
　　public boolean isValid(String word) {･･･}<br>
　　public List＜String＞ suggestions(String typo) {･･･}<br>
}<br>

#### 4. まとめ
* クラスが下層の資源に依存する場合、シングルトンや静的なユーティリティクラスは使用しない。
* 資源もしくは資源を生成するファクトリをコンストラクタ or staticファクトリ or ビルダーに渡す。<br>
<br>

### 項目6. 不必要なオブジェクトの生成は避ける
#### 1. Stringクラス
>　String s = new String("money");<br>

実行されるごとに不要なStringインスタンスが新たに生成されるため、<br>
>　String s = "money";<br>

と記載することで、一つのStringインスタンスを使用し、オブジェクトを再利用する。

#### 2. booleanクラス
>  {<br>
　String booleanString = "true";<br>
　Boolean b = Boolean(booleanString); // オブジェクト生成<br>
}

>  {<br>
　String booleanString = "true";<br>
　Boolean b = Boolean.valueOf(booleanString); // 新たなオブジェクトを生成しない<br>
}

<br>

### 項目7. 使われなくなったオブジェクト参照を取り除く
メモリリークを抱えてると、稀にページングを引き起こしたり、OutOfMemoryErrorを引き起こすことがある。また、パフォーマンスが低下する。<br>

#### 1. メモリリークしやすいとき
* クラスが独自のメモリを管理しているとき
* キャッシュ
* リスナーやコールバック

#### 2. キャッシュへの対処 
* WeakHashMapで実装する<br>
⇒キャッシュエントリの生存期間が、キーへの外部からの参照によって決定する場合に有効。
* TimerやScheduledThreadPoolExecutor（バックグラウンドのスレッド）を用いる<br>
⇒時間の経過によって適時除去する。
* LinkHashMapクラスで実装する<br>
⇒登録したタイミングが一番古いものを除去する。

#### 3. リスナーやコールバック 
* WeakHashMapで実装する<br>
<br>

### 項目8. ファイナライザとクリーナーを避ける
ファイナライザ（finalizar()メソッド）とは、<br>

ファイナライザとC++のデストラクタは違うもの。<br>
⇒Javaでデストラクタに当たるのは、ファイナライザではなくtry-finallyブロック

#### 1. ファイナライザの欠点 
* 即時でファイナライザが実行される保証がない<br>
    1. ガーベジコレクションの機能であるため、JVMの実装に依存する
    1. オブジェクトの回収が通常より遅れる場合がある
    1. ファイナライザが実行される保証自体ない

* ファイナライザでスローされた例外がキャッチされなかった場合、ファイナライザが単に終了する

* 処理が遅い

#### 2. まとめ
* セーフティネットあるいは重要でないネイティブ資源を解放させる以外でファイナライザは使用しない。
* セーフティネットとしての使用時はエラーログを出力する<br>
<br>

### 項目9. try-finallyよりもtry-with-resourcesを選ぶ
歴史的にはtry-finally分が、例外やリターンに際して、資源の適切なクローズを保証していた。
> static String firstLineOfFile(String path) throws IOException {<br>
　　BufferedReader br = new BufferedReader(new FileReader(path));<br>
　　try {<br>
　　　　retrun br.readLine();<br>
　　} finally {<br>
　　　　br.close();<br>
　　}<br>
}

上記は悪くないように見えるが、二つ目の資源を加えると悪化する。
> static void copy(String src, String dst) throws IOException {<br>
　　InputStream in = new FileInputStream(src);<br>
　　try {<br>
　　　　OutputStream out = new FileOutputStream(dst);<br>
　　　　try {<br>
　　　　　　byte[] buf = new byte[BUFFER_SIZE];<br>
　　　　　　int n;<br>
　　　　　　while ((n = in.read(buf)) ＞= 0)<br>
　　　　　　　　out.write(buf, 0, n);<br>
　　　　} finnaly {<br>
　　　　　　out.close();<br>
　　　　}<br>
　　} finally {<br>
　　　　in.close();<br>
　　}<br>
}

tryブロックとfinallyブロック内のコードは、いずれも例外をスローできる。<br>
* firstLineOfFileメソッドでは、readLineの呼び出しが下層の物理デバイスでの失敗により例外をスローする可能性があり、同じ理由でcloseも失敗する可能性がある。<br>
⇒このような状況下では、二つ目の例外は一つ目の例外を覆い隠してしまっている。例外のスタックトレースには税所の例外の記録はなく、実際のシステムでのデバッグを複雑にしてしまう。<br>
上記問題を解決するには、try-with-resourcesを使用する。<br>
> static String firstLineOfFile(String path) throws IOException {<br>
　　try ( BufferedReader br = new BufferedReader(<br>
　　　　　　new FileReader(path))) {<br>
　　　　retrun br.readLine();<br>
} 

と
> static void copy(String src, String dst) throws IOException {<br>
　　try ( InputStream in = new FileInputStream(src);<br>
　　　　OutputStream out = new FileOutputStream(dst)) {<br>
　　　　　　byte[] buf = new byte[BUFFER_SIZE];<br>
　　　　　　int n;<br>
　　　　　　while ((n = in.read(buf)) ＞= 0)<br>
　　　　　　　　out.write(buf, 0, n);<br>
　　}<br>
} 

とする。catch節を持つtry-with-resourcesは<br>
> static String firstLineOfFile(String path, String defaultVal) {<br>
　　try ( BufferedReader br = new BufferedReader(<br>
　　　　　　new FileReader(path))) {<br>
　　　　retrun br.readLine();<br>
　　catch (IOException e) {<br>
　　　　return defaultVal;
　　}<br>
} 

<br>

## 2. すべてのオブジェクトに共通のメソッド
### 項目10.equalsをオーバーライドするときは一般契約に従う 
#### 1. equalsをオーバーライドしないときの条件
* クラスの個々のインスタンスが本質的に一意のとき<br>
例）Threadクラス
* クラスが「論理的等価性（logical equality）」の検査を提供する必要がないとき<br>
例）java.util.Randomクラス
* スーパークラスがすでにequalsをオーバーライドしており、スーパークラスの振る舞いがクラスに対して適切であるとき<br>
例）ほとんどのSetの実装はAbstractSetからequalsの実装を継承している。ほとんどのListはAbstractList、ほとんどのMapはAbstractMapから継承。
* クラスがprivateあるいはパッケージプライベートであり、そのequalsメソッドが呼び出されないことが確かであるとき<br>

#### 2. equalsをオーバーライドするべきときの条件
* クラスが論理等価性という概念を持っており、かつ、スーパークラスがequalsをオーバーライドしていないとき<br>
⇒値クラスを指すことが多い。<br>
例）Integer、Date

#### 3. 一般契約
equalsをオーバーライドするときは一般契約を順守しなければならない。<br>
* 反射的
* 対照的
* 推移的
* 整合的
* 非null

#### 4. 反射的
普通に実装していれば特に気にする必要はない。

#### 5. 対象的
簡単に破ってしまう可能性があるため、注意が必要。
具体例を記載する。

#### 6. 推移的
簡単に破ってしまう可能性があるため、注意が必要。
具体例を記載する。
<br>
<br>

### 項目11. 
※デザインパターンについて学習が必要。
そのうえでもう一回Effective Javaをやる必要がある。
