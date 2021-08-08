##Covariant, Contravariant, Invariant in Java Generic
因為最近正在讀 [Java Generic and Collections](https://www.oreilly.com/library/view/java-generics-and/0596527756/) 這本書，其中某些地方真的獲益良多，在這裡寫一篇關於 Covariant, Contravariant, Invariant 概念的簡短介紹（ 這篇不涉及 Generic Class 的設計 ）。
要了解這些概念，剛開始最重要的是了解什麼是 Subtype，在 Java，一個可以判斷物件 A 的 type 是否為物件 B 的 type 的 subtype 的最簡單方法，就是 `B = A;` 這個 expression 是否合法 ( 有辦法 compile )，只要滿足這個條件，便可以說 A 的 type 是 B 的 type 的 subtype，而 B 是 A 的 supertype。
```java
Integer a = 1; //合法，故 Integer 為 Integer 的 Subtype
Number b = a; //合法，故 Integer 為 Number 的 Subtype
Integer c = b; //不合法(compile error)，故 Number 不為 Integer 的 Subtype
a = null; //合法，故 null 為 Integer 的 Subtype
Object d = a; //合法，故 Integer 為 Object 的 Subtype
```
由上面可知，任何 type 為自己的 subtype，`null` 為任何 type 的 subtype，`Object` 為任何 type 的 supertype。
講到這裡，跟這篇要提到的有什麼關係？這裡是三種 Variant 的定義（以下 A ≤ B 代表 A 為 B 的 Subtype ），其中 f 代表型別轉換，傳入一種 Type，得到另一種 Type：

- **Covariant** : 若 A ≤ B 則 f(A) ≤ f(B) 
- **Contravariant**：若 A ≤ B 則 f(B) ≤ f(A) 
- **Invariant**：上述兩項都不成立

上述的 f 代表什麼？以 Array 舉個例子，假設 f 為一個 function 名為 Arrayof，Arrayof(Integer) 就代表 `Integer[]`，那我們知道Integer為Object的Subtype，Arrayof(Integer) 與 Arrayof(Object) 的關係又是如何？在 Java，`Integer[]` 為 `Object[]` 的 Subtype，故為 Covariant 。
```java
Integer[] nums = new Integer[1];
Object[] objs = nums; //合法，故 Integer[] 為 Object[] 的Subtype
objs[0] = 1.5 //合法，但其中 1.5 為 Double，然而此 Array 真的 type 為 Integer[]，故在 'Runtime' 會噴出 array store exception
```
由上可以知道 Covariant 的基本用法，也可以發現 Java Array 中設計的缺陷，導致第三行的錯誤要在 Runtime 才抓得到，這主要是因為 Java 初期還沒有 Generic ，又必須為 Array 滿足 Covariant 的需求，才如此設計。
Array 的例子做完了，那 List 呢？ Java 中 List 是設計成 Generic 的，我能夠在 class 標註其他 type 來限制 class 使用的 type，例如 `List<String>` 就限制了此類別的 List 只能夠儲存 String type的物件，上面的Arrayof 改成 Listof，Listof(String) 就代表 `List<String>`，以 `Number` 及 `Integer` 來舉例，`List<Integer>` 不為 `List<Number>` 的 subtype，`List<Number>` 也不為 `List<Integer>` 的 subtype，故為 Invariant。
```java
List<Integer> names = new ArrayList<Integer>(); //合法，ArrayList 為 List 的 subtype
List<Number> objs = names; //不合法
```
因為是 Invariant，所以第二行無法 compile，所以就不會發生 Array 在 runtime 才噴出 exception 的問題，那...... List 就無法以 Covariant 的方式使用了嗎？答案是可以的。
官方提供了額外的語法將 List 定義支援 Covariant，假設是以上述的例子，我可以把第二行的宣告改寫成 `List<? extends Number>` ，這代表若 type A 是 Number 的 subtype，則 `List<A>` 為 `List<? extends Number>` 的 subtype，其中 <? extends Number> 代表任何 Number 的 subtype。
```java
List<Integer> ints = Arrays.asList(1);
List<Double> doubles = Arrays.asList(1.1);
List<Float> floats = Arrays.asList(1.1f);
List<? extends Number> nums = ints; //合法
nums = doubles; //合法
nums = floats; //合法
Number num = nums.get(0); // 合法
nums.add(1); //不合法
nums.add(null); //合法
```
第七行之所以合法是因為該行若要能 compile ，則代表左要為右的 supertype， 就是必須要是 <? extends Number> 的 supertype ，而任何 `Number` 的 subtype 的 supertype 就是 `Number` 所以合法，然而第八行，傳入的物件 type 必須要是 <? extends Number> 的 subtype，簡單來說就是任何 Number 的 subtype 的 subtype，除了第九行的 null（任何 type 的 subtype） ，沒有其他合法的寫法，因此 covariant 的 List 適用情況只有在從 List 讀取值的時候。
那當要從 List 寫入值的時候該怎麼做？這時就是 Contravariant 適用的情況，在 List 上的寫法是宣告成 `List<? super Integer>`，如此宣告代表若 type A 是 `Integer` 的 supertype，則 `List<A>` 為 `List<? super Integer>` 的 subtype，其中? super Integer 是代表任何 `Integer` 的 supertype。
```java
List<Integer> ints = Arrays.asList(1);
List<Number> nums = Collections.emptyList();
List<Object> objs = Arrays.asList(new Object());
List<? super Integer> intList = ints; //合法
intList = nums; //合法
intList = objs; //合法
intList.add(1); //合法
Integer number = intList.get(0); //不合法
Object obj = intList.get(0); //合法
```
可以看到第七行，能夠合法的寫入 1 ( Integer ) ，因為 <? super Integer> 的 subtype 代表任何 `Integer` 的 supertype 的 subtype，那就是 `Integer`，然而 get 出來得到的值必須要是 <? super Integer> 的 supertype，也就是任何 `Integer` 的 supertype 的 supertype，所以第八行是不合法的，只有第九行的 `Object` ( 任何物件的 supertype ) 才合法，所以可以了解到 Contravariant 適用的情況是寫入值。
上述的技巧是給 Generic type 加上 Bound，基本上使用 Java 的 Generic 設計程式的原則就是，能用到 Bound 就盡量用，這樣便可以增加程式碼的活用度，假設有以下程式碼：
```java
public static <T> void copy(List<T> dest, List<T> src){
  int srcSize = src.size();
  if(srcSize > dest.size())
    throw new IndexOutOfBoundsException();
  for (int i=0; i<srcSize; i++)
    dest.set(i, src.get(i));
} 
```
上述的程式碼較偏向是 util method，將傳入的 src List 內的值都 copy 到 dest List內，並沒有用到任何的 Bound ，所以我呼叫的地方就只能指定兩個 List 為同一個單一型態的 List，像是：
```java
List<Integer> src = Arrays.asList(4, 5, 6);
List<Integer> dest = Arrays.asList(1, 2, 3);
copy(dest, src);
assert dest.equals(Arrays.asList(4, 5, 6));
List<Number> dest2 = new ArrayList<Number>(3);
copy(dest2, src); //不合法
```
若改用 Bound 則改寫成：
```java
                                 /*寫入*/                /*讀取*/
public static <T> void copy(List<? super T> dest, List<? extends T> src){
  int srcSize = src.size();
  if(srcSize > dest.size())
    throw new IndexOutOfBoundsException();
  for (int i=0; i<srcSize; i++)
    dest.set(i, src.get(i));
} 
//在Java Collections class，可以找到此 method 最完整的 implementation，此為示範用
```
如此寫法就增加了活用度：
```java
List<Integer> ints = Arrays.asList(1,2);
List<Double> doubles = Arrays.asList(1.1,2.2);
List<Float> floats = Arrays.asList(1.1f,2.2f);
List<Number> nums = new ArrayList<Number>(2);
copy(nums,ints);
copy(nums,doubles);
copy(nums,floats);
```
由於上述解釋 List Covariant 及 Contravariant 分別適用於讀取值及回傳值的情況，有可能讓人誤認為這只是 List 的特性，事實上這特性是所有 Generic class 都適用的，先看看這一段程式碼：
```java
public Integer doubleValue(Number num){
  return 2 * num.intValue();
}

public static void main(String[] args){
  Integer integer = doubleValue(1.1); //合法，傳入 Double
  Number number = doubleValue(1.1f); //合法，傳入 Float
  Object object = doubleValue(1); //合法，傳如 Integer
}
```
可以看到，上述main中能夠合法呼叫的前提是：**傳入的 parameter 至少要是 signature 定義 type 的subtype，而接收 return 值的物件至少要是 signature return type 的 supertype**，這個定義有另外一個常用的簡寫 **PECS (Producer Extends Consume Super)**，這定義適用於任何method。
然後看看這段 List 的程式碼：
```java

public interface List<E> extends Collection<E> {
  /*.
    .省略，完整請見 Java List source code
    .*/
                    /*傳入*/
  public boolean add(E object);
        /*回傳*/
  public E       get(int location);
}
```
可以看到 method add 的定義是要傳入 generic type 的物件，而 get 則是回傳 generic type 的物件，若這時使用 covariant 宣告 List 則 signature變成：
```java
public boolean add(<? extends E> object);
public <? extends E> get (int location);
```
再加上上面粗體字所寫的，就會知道為什麼 List 在 covariant 的狀況只能讀值，而同樣的，寫成 contravariant 的狀況：
```java
public boolean add(<? super E> object);
public <? super E> get (int location);
```
就會知道為什麼 contravariant 的狀況只能寫入值。
所以總結來說，在使用 generic class 時，若要使用的 method 是需要取出 generic type 的物件時，使用 Covariant，若是傳入 generic type 的物件時，則使用 Contravariant ，若都需要用到，就只能用 Invariant。 