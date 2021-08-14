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
`nums = [1, 2, 2, 4, 5, 7], target = 3`
|L| |M| | |H|
|-|-|-|-|-|-|
|1|2|2|4|5|7|
↓
| | |L|M| |H|
|-|-|-|-|-|-|
|1|2|2|4|5|7|
↓
| | |L/M|H| | |
|-|-|-  |-|-|-|
|1|2|2  |4|5|7|
可以看到最後一步開始無窮迴圈了，因為在最後一步時 `(High + Low) / 2` 永遠都等於 2 `//TODO`
以上的 Binary Search 實作其實藏有一些問題，但如果對於 Binary Search 沒有很好的理解可能不會發現有問題，又或著是察覺得到有問題，但可能要實際上丟幾組測試資料去跑跑看才能發現出了問題，然後在對應的地方補個 +1 或 -1 去設法解掉，然而這種做法一是可能沒有解決完全部可能出現的問題，二是本身還是對於問題的本質不了解。
## 搜尋區間
Binary Search 本質上就是不斷的將結果的可能出現區間刪去掉已經判斷為不可能的那一半，最後在不段縮小的區間內找到答案，而我認為大多數寫出問題的 Binary Search 程式的問題出在**沒有在刪除區間時確實把所有已判斷為可刪除的值從區間刪除**，從而導致無窮迴圈。
### 搜尋區間的定義方式
最常看到的搜尋區間定義方式應該就這兩種：
|    程式碼   |   區間定義   |              起始值              |
|-------------|--------------|----------------------------------|
|`low <= high`| `[low, high]`|`low = 0, high = array.length    `|
|`low < high` | `[low, high)`|`low = 0, high = array.length - 1`|


|l|1|m|3|4|h|
|-|-|-|-|-|-|
|1|2|2|~~4~~|~~**5**~~|~~7~~|

