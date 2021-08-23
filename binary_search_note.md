## Binary Search 心得
Binary Search 是一個是一個很基礎的演算法，用於在以排序過的陣列內找尋資料，能達到 log(n) 的時間複雜度。一個最基礎的 Binary Search function 如下：
```java
int binarySearch(int[] nums, int target) {
  int low = 0;
  int high = nums.length - 1;
  while (low < high) {
    int mid = (high - low) / 2 + low;
    if (nums[mid] > target) {
      high = mid;
    } else if (nums[mid] == target) {
      return mid;
    } else {
      low = mid + 1;
    }
  }
  return -1;
}
```
因為傳入的 `nums` 是已排序的陣列，所以便可以每次取中間 `mid` index 的值來每次將目標得值可能的所在範圍縮減一半，又因為每次都是除以 2，所以最大的搜尋次數就是 log(n)。
以上的邏輯看起來簡單也很好理解，然而若不小心處理很容易遇到 **off by one** 的問題，其中我讀過[這篇](https://kkc.github.io/2019/03/28/learn-loop-invariant-from-binary-search/)文章之後我認為我開始有方向對於 Binary Search 的本質有更好的理解，我自己也有些心得，先看看以下的程式碼吧：
```java
int binarySearch(int[] nums, int target) {
  int low = 0;
  int high = nums.length;
  while (low <= high) {
    int mid = (high - low) / 2 + low;
    if (nums[mid] > target) {
      high = mid;
    } else if (nums[mid] == target) {
      return mid;
    } else {
      low = mid;
    }
  }
  return -1;
}
```
以上程式碼邏輯可能看似沒什麼問題，但我們下面模擬看看

`nums = [1, 2, 2, 4, 5, 7], target = 3`：

<svg xmlns="http://www.w3.org/2000/svg" width="244" height="84">

 <g stroke-width="2" stroke="#fff" fill="none">
  <text y="25%" alignment-baseline="central" text-anchor="middle"  x="22">Low</text>
  <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">Mid</text>
  <text y="25%" alignment-baseline="central" text-anchor="middle" x="222">High</text>
 </g>
 <g transform="translate(0,40)" stroke="#fff" stroke-width="4" fill="none">
  <rect height="40" width="40" y="2" x="2"/>
  <rect height="40" width="40" y="2" x="42"/>
  <rect height="40" width="40" y="2" x="82"/>
  <rect height="40" width="40" y="2" x="122"/>
  <rect height="40" width="40" y="2" x="162"/>
  <rect height="40" width="40" y="2" x="202"/>
  <rect height="40" width="40" y="2" x="2" stroke="#006400"/>
  <rect height="40" width="40" y="2" x="202" stroke="#006400"/>
  <rect height="40" width="40" y="2" x="82" stroke="#006400"/>
 </g>
 <g  transform="translate(0,40)" stroke-width="2" stroke="#fff" fill="none">
    <text y="25%" alignment-baseline="central" text-anchor="middle"  x="22">1</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="62">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="142">4</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="182">5</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="222">7</text>
  </g>
</svg>

↓

<svg xmlns="http://www.w3.org/2000/svg" width="244" height="84">

 <g stroke-width="2" stroke="#fff" fill="none">
  <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">Low</text>
  <text y="25%" alignment-baseline="central" text-anchor="middle"  x="142">Mid</text>
  <text y="25%" alignment-baseline="central" text-anchor="middle" x="222">High</text>
 </g>
 <g transform="translate(0,40)" stroke="#fff" stroke-width="4" fill="none">
  <rect height="40" width="40" y="2" x="2"/>
  <rect height="40" width="40" y="2" x="42"/>
  <rect height="40" width="40" y="2" x="82"/>
  <rect height="40" width="40" y="2" x="122"/>
  <rect height="40" width="40" y="2" x="162"/>
  <rect height="40" width="40" y="2" x="202"/>
  <rect height="40" width="40" y="2" x="122" stroke="#006400"/>
  <rect height="40" width="40" y="2" x="202" stroke="#006400"/>
  <rect height="40" width="40" y="2" x="82" stroke="#006400"/>
 </g>
 <g  transform="translate(0,40)" stroke-width="2" stroke="#fff" fill="none">
    <text y="25%" alignment-baseline="central" text-anchor="middle"  x="22">1</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="62">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="142">4</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="182">5</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="222">7</text>
  </g>
</svg>

↓

<svg xmlns="http://www.w3.org/2000/svg" width="244" height="84">

 <g stroke-width="2" stroke="#fff" fill="none">
  <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">L/M</text>
  <text y="25%" alignment-baseline="central" text-anchor="middle"  x="142">High</text>
 </g>
 <g transform="translate(0,40)" stroke="#fff" stroke-width="4" fill="none">
  <rect height="40" width="40" y="2" x="2"/>
  <rect height="40" width="40" y="2" x="42"/>
  <rect height="40" width="40" y="2" x="82"/>
  <rect height="40" width="40" y="2" x="122"/>
  <rect height="40" width="40" y="2" x="162"/>
  <rect height="40" width="40" y="2" x="202"/>
  <rect height="40" width="40" y="2" x="122" stroke="#006400"/>

  <rect height="40" width="40" y="2" x="82" stroke="#006400"/>
 </g>
 <g  transform="translate(0,40)" stroke-width="2" stroke="#fff" fill="none">
    <text y="25%" alignment-baseline="central" text-anchor="middle"  x="22">1</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="62">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="102">2</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="142">4</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="182">5</text>
    <text y="25%" alignment-baseline="central" text-anchor="middle" x="222">7</text>
  </g>
</svg>

可以看到最後一步開始無窮迴圈了，因為在最後一步時 `(High + Low) / 2` 永遠都等於 2，那要如何修改才是對的呢？是把 `high = mid` 改成 `high = mid - 1`，還是把 `low = mid` 改成 `low = mid + 1`，還是都改？有時候可能運氣好猜對了但對於這件事的本質沒有很好的理解，而且也有可能只是在當前這個 case 對了，我認為理解搜尋區間的定義就可以更輕鬆的判斷如何寫才是對的。
## 搜尋區間
Binary Search 本質上就是不斷的將結果的可能出現區間刪去掉已經判斷為不可能的那一半，最後在不段縮小的區間內找到答案，而我認為大多數寫出問題的 Binary Search 程式的問題出在**沒有在刪除區間時確實把所有已判斷為可刪除的值從區間刪除**，從而導致無窮迴圈。
### 搜尋區間的定義方式
最常看到的搜尋區間定義方式應該就這兩種：
<table>
  <tr>
    <td>程式碼</td>
    <td>區間定義</td>
    <td>起始值 </td>
  </tr>
  <tr>
    <td>low <= high</td>
    <td>[low, high]</td>
    <td>low = 0, high = array.length</td>
  </tr>
  <tr>
    <td>low < high</td>
    <td>[low, high)</td>
    <td>low = 0, high = array.length - 1</td>
  </tr>
</table>

|程式碼|區間定義|起始值|
|---|---|---|
|low <= high|[low, high]|low = 0, high = array.length|
|low < high|[low, high)|low = 0, high = array.length - 1|