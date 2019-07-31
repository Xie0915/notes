### 162 [寻找峰值](https://leetcode-cn.com/problems/find-peak-element/)

描述：给定一个输入数组 `nums`，其中 `nums[i] ≠ nums[i+1]`，找到峰值元素并返回其索引。

**==思路==**：二分查找，查找到的数与相邻的下面一个数进行比较。比下一个数小则在之前的区间，比下一个数大则在之后的区间

```java
public int findPeakElement(int[] nums){
    int lo = 0, hi = nums.length - 1;
    while (lo < hi){
        int mid = (lo + hi) / 2;
        if (nums[mid] < nums[mid + 1]) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```



### 165  [比较版本号](<https://leetcode-cn.com/problems/compare-version-numbers/comments/>)

描述：比较两个版本号，版本号分级，用'.'区别

==思路==：用Java的string.split函数分开就好

```java
public int compareVersion(String version1, String version2) {
        String[] componentOfVer1 = version1.split("[.]");
        String[] componentOfVer2 = version2.split("[.]");
        int index = 0;
        while (index < componentOfVer1.length && index < componentOfVer2.length){
            if (Integer.parseInt(componentOfVer1[index]) < Integer.parseInt(componentOfVer2[index])) return -1;
            else if (Integer.parseInt(componentOfVer1[index]) > Integer.parseInt(componentOfVer2[index])) return 1;
            else index++;
        }
        if (index < componentOfVer1.length){
            while (index < componentOfVer1.length){
                if (Integer.parseInt(componentOfVer1[index]) != 0) return 1;
                index++;
            }
        }
        else if (index < componentOfVer2.length){
            while (index < componentOfVer2.length){
                if (Integer.parseInt(componentOfVer2[index]) != 0) return -1;
                index++;
            }
        }
        return 0;
    }
```

Java知识点：

- String.split 在分割特殊符号时，以下对应：

|  .   | \\\\\. |
| :--: | :----: |
|  \|  | \\\\|  |

- Java String 转 Int：

```java
Integer.parseInt(String s);
Integer.valueOf(String s).intValue()
```



### 166 [分数转小数](https://leetcode-cn.com/problems/fraction-to-recurring-decimal/comments/)

描述：给定分子分母，转成小数，如果有循环，用括号表示

==思路==：其他还挺简单，考虑负数，-2147483648，不是从第一位开始循环啥的，写起来有点烦

```java
public String fractionToDecimal(int numerator, int denominator) {
        HashMap<Long, Integer> dividend = new HashMap<>();
        StringBuffer resultBuffer = new StringBuffer();
        long dividendNum = numerator;
        long toBeDividedNum = denominator;
        if (dividendNum < 0 ^ toBeDividedNum < 0 && dividendNum != 0){
            resultBuffer.append('-');
        }
        dividendNum = Math.abs(dividendNum);
        toBeDividedNum = Math.abs(toBeDividedNum);
        resultBuffer.append(dividendNum / toBeDividedNum);
        if (dividendNum % toBeDividedNum == 0){
            return resultBuffer.toString();
        }
        resultBuffer.append('.');
        int offset = resultBuffer.length();
        int occurLocation = 0;
        dividendNum = dividendNum % toBeDividedNum;
        while (dividendNum != 0 && !dividend.containsKey(dividendNum)){
            dividend.put(dividendNum, occurLocation++);
            resultBuffer.append(dividendNum * 10 / toBeDividedNum);
            dividendNum = (dividendNum * 10) % toBeDividedNum;
        }
        if (dividend.containsKey(dividendNum)){
            occurLocation = dividend.get(dividendNum);
            resultBuffer.insert(offset + occurLocation, '(');
            resultBuffer.append(')');
            return resultBuffer.toString();
        }
        else return resultBuffer.toString();
    }
```

Java知识点：

- HashMap：

  ```java
  HashMap.get(key)
  HashMap.put(key, value)
  HashMap.containsKey(key)
  HashSet.add(key)
  HashSet.contains(key)
  ```

- StringBuffer：

  ```java
  StringBuffer.append()
  StringBuffer.insert()
  StringBuffer.length()
  StringBuffer.toString()
  ```



### 167 [两数之和2](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/comments/)

描述：给定的为有序数组，返回两个和为目标值的数的索引下标

==思路==：两个指针，从两边向中间扫描

```java
public int[] twoSum(int[] numbers, int target) {
    int lo = 0, hi = numbers.length - 1;
    while (lo < hi && numbers[lo] + numbers[hi] != target){
        if (numbers[lo] + numbers[hi] < target) lo++;
        else hi--;
    }
    return new int[] {lo + 1, hi + 1};
}
```

Java知识点：

- 数组的初始化方式：

  ```java
  return new int[] {lo + 1, hi + 1};
  ```



### 168 [Excel表](https://leetcode-cn.com/problems/excel-sheet-column-title/submissions/)

描述：将数字变成EXCEL表中的名称，如1-A, 27-AA, 28-AB

==思路==：取26的模和商即可

```java
public String convertToTitle(int n) {
    if (n <= 0) return null;
    StringBuffer resultBuffer = new StringBuffer();
    while (n != 0) {
        n--;
        resultBuffer.append((char)(n % 26 + 'A'));
        n /= 26;
    }
    return resultBuffer.reverse().toString();
}
```

Java知识点：

- StringBuffer的reverse：

  ```java
  return resultBuffer.reverse().toString();
  ```

- char型强制类型转换以及ascii计算：

  ```java
  resultBuffer.append((char)(n % 26 + 'A'));
  ```




### 171 [Excel表序号](https://leetcode-cn.com/problems/excel-sheet-column-number/submissions/)

描述：将表名称变为数字，如 AA-27

==思路==：比较简单，注意乘26就行了

```java
public int titleToNumber(String s) {
    int result = 0;
    int pos = 1;
    for (int idx = s.length() - 1; idx >= 0 ; idx--) {
        char charToHandle = s.charAt(idx);
        if (charToHandle < 'A' || charToHandle > 'Z') return -1;
        result += (charToHandle - 'A' + 1) * pos;
        pos *= 26;
    }
    return result;
}
```



### 172 [阶乘后的0](https://leetcode-cn.com/problems/factorial-trailing-zeroes/)

描述：找到阶乘后的0的总数

思路：找出5的个数

```java
public int trailingZeroes(int n) {
    int result = 0;
    while(n >= 5){
        result += n / 5;
        n /= 5;
    }
    return result;
}
```



### 173 [二叉搜索迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/submissions/)

描述：实现一个二叉搜索树的迭代器，包含next(), hasNext()方法

==思路==：用栈，将父节点保存在栈中，注意处理特殊情况，比如root为0

```java
class BSTIterator {
    private TreeNode currentNode = null;
    private Stack<TreeNode> nodeNotTraverse = new Stack<>();
    public BSTIterator(TreeNode root) {
        if (root == null) currentNode = null;
        else {
            while (root.left != null){
                nodeNotTraverse.push(root);
                root = root.left;
            }
            nodeNotTraverse.push(root);
        }
    }
    /** @return the next smallest number */
    public int next() {
        if (currentNode == null){
            currentNode = nodeNotTraverse.pop();
            return currentNode.val;
        }
        if (currentNode.right != null) {
            currentNode = currentNode.right;
            while (currentNode.left != null){
                nodeNotTraverse.push(currentNode);
                currentNode = currentNode.left;
            }
            return currentNode.val;
        }
        else {
            currentNode = nodeNotTraverse.pop();
            return currentNode.val;
        }
    }
    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        if (!nodeNotTraverse.isEmpty() || (currentNode != null && currentNode.right != null)) return true;
        else return false;
    }
}
```

Java知识点：

定义一个引用对象，比如栈时，在使用前一定要先在堆上new一个出来，下述使用

```java
Stack <Integer> stack;
stack.push(5);
```

将报 NullPointerException 错误



### 179 [最大数](<https://leetcode-cn.com/problems/largest-number/submissions/>)

描述：一组非负整数，重新排列以获得最大的整数

==思路==：比较 s1+ s2 与 s2 + s1的大小，注意整数越界以及全为0的情况

```java
// 1 用 Collections.sort
StringBuilder result = new StringBuilder();
List<String> numsInStrForm = new ArrayList<>();
for (int i : nums) numsInStrForm.add(String.valueOf(i));
Collections.sort(numsInStrForm, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        String s1 = o1 + o2;
        String s2 = o2 + o1;
        if (Long.parseLong(s1) < Long.parseLong(s2)) return 1;
        else if (Long.parseLong(s1) == Long.parseLong(s2)) return 0;
        else return -1;
    }
});
if (numsInStrForm.get(0).equals("0")) return "0";
for (String str : numsInStrForm) result.append(str);
return result.toString();


// 2 用Arrays.sort
public String largestNumber(int[] nums) {
    StringBuilder result = new StringBuilder();
    String [] numsInStrForm = new String[nums.length];
    for (int i = 0; i < nums.length; i++) {
        numsInStrForm[i] = String.valueOf(nums[i]);
    }
    Arrays.sort(numsInStrForm, new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            String s1 = o1 + o2;
            String s2 = o2 + o1;
            if (Long.parseLong(s1) < Long.parseLong(s2)) return 1;
            else if (Long.parseLong(s1) == Long.parseLong(s2)) return 0;
            else return -1;
        }
    });
    if (numsInStrForm[0].equals("0")) return "0";
    for (String str : numsInStrForm) result.append(str);
    return result.toString();
}
```

Java知识点：

定义比较器的时候，只能定义为引用对象

Arrays.sort输入为数组对象，而Collections.sort输入为实现了list接口的对象

int转string

```java
String.valueOf(int i)
```

