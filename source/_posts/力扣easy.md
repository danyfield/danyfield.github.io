---
title: 力扣easy
date: 2023-02-13 21:09:27
tags: "算法"
categories: "算法"
---

### 二分查找

输入是一个有序的元素列表；最多需要检查`log n`个元素

```go
func search(num []int, target int) int {
    // 有序查找可以考虑使用二分法
    //初始化left和right,即最左侧和最右侧位置
    left := 0
    right := len(nums) - 1
    
    //循环条件left<right
    for left <= right {
        mid := (left + right)/2
        if num[mid] == target {
            return mid
        } else if num[mid] > target {
            right = mid - 1
        } else {
            left = mid + 1
        }
    }
    
    return -1
}
```

- 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引，若目标值不存在于数组中，返回它将会被插入的位置

   ```
   示例1：
   	输入：nums = [1,3,5,6],target = 5
   	输出：2
   示例2：
   	输入：nums = [1,3,5,6],target = 2
   	输出：1
   示例3：
   	输入：nums = [1,3,5,6],target = 7
   	输出：4
   ```

   ```go
   func searchInsert(nums []int, target int) int {
   	/*
   		第一部分：使用二分法确定位置
   		第二部分：若不存在则插入到指定位置，每次和中间值比较，当到最后一轮时，若大于则为mid+1，小于则为mid-1
   	*/
   	left := 0
   	right := len(nums) - 1
   	tag := 0
   
   	for left <= right {
   		mid := (left + right) / 2
   		if nums[mid] == target {
   			return mid
   		} else if nums[mid] < target {
   			left = mid + 1
   			tag = left
   		} else {
   			right = mid - 1
   			tag = mid
   		}
   	}
   
   	return tag
   }
   ```

### 哈希表

- `两数之和：`给定一个整数数组`nums`和一个整数目标值`target`，找出和为目标值的两个整数并返回它们的数组下标（可以假定只有一个答案，但同一个元素不能重复出现）

  ```
  示例1：
  	输入：nums = [2,7,11,15],target = 9
  	输出：[0,1]
  示例2：
  	输入：nums = [3,2,4],target = 6
  	输出：[1,2]
  示例3：
  	输入：nums = [3,3],target=6
  	输出：[0,1]
  ```

  ```go
  func twoSum(nums []int,target int) []int {
      //定义一个哈希表存放自己的位置，方便其他值寻找
      mapList := make(map[int]int,len(nums))
      for i := 0; i < len(nums); i++ {
          //若哈希表中存在自己想要的值则返回
          if _,ok := mapList[nums[i]]; ok {
              return []int{mapList[nums[i]],i}
          }
          mapList[target - nums[i]] = i
      }
      
      return []int{}
  }
  ```

  ```php
  class Solution {
      /**
       * @param Integer[] $nums
       * @param Integer $target
       * @return Integer[]
       */
      function twoSum($nums, $target) {
          //定义一个哈希表数组
          $tmp = [];
          //定义一个存放返回结果的数组
          $arr = [];
          foreach($nums as $key=>$value){
              if(isset($tmp[$target - $value])){
                  $arr = [$tmp[$target - $value],$key];
                  break;
              }
              $tmp[$value] = $key;
          }
          
          return $arr;
      }
  }
  ```

- `罗马数字转整数`

  ```
  示例1：
  	输入：s = "III"
  	输出：3
  示例2：
  	输入：s = "IV"
  	输出：4
  示例3：
  	输入：s = "IX"
  	输出：9
  示例4：
  	输入：s = "LVIII"
  	输出：58
  ```

  ```go
  func romanToInt(s string) int {
      // 定义一个哈希表存储罗马字符的值
      roman := map[byte]int{
          'I':1,
          'V':5,
          'X':10,
          'L':50,
          'C':100,
          'D':500,
          'M':1000,
      }
  
      //num存储总数，从右至左遍历
      num := roman[s[len(s)-1]]
  
      for i := len(s)-2; i >= 0; i-- {
          if roman[s[i]] >= roman[s[i+1]] {
              num += roman[s[i]]
          } else {
              num -= roman[s[i]]
          }
      }    
  
      return num
  }
  ```

  ```php
  class Solution {
      /**
       * @param String $s
       * @return Integer
       */
      function romanToInt($s) {
          $arr = ['I'=>1,'V'=>5,'X'=>10,'L'=>50,'C'=>100,'D'=>500,'M'=>1000];
          //记录结果
          $res = 0;
          for($i=0;$i<strlen($s);$i++){
              if(!isset($arr[$s[$i]]) || $arr[$s[$i]] >= $arr[$s[$i+1]]){
                  $res += $arr[$s[$i]];
              } else {
                  $res -= $arr[$s[$i]];
              }
          }
          return $res;
      }
  }
  ```

### 字符串

- `回文数：`判断一个整数是否为回文数

  ```
  示例1：
  	输入：x = 121
  	输出：true
  示例2：
  	输入：x = -121
  	输出：false
  示例3：
  	输入：x = 10
  	输出：false
  ```

  ```go
  func isPalindrome(x int) bool {
      //若值为负数或以0结尾但不是0
      if x < 0 || (x % 10 == 0 && x != 0)	return false
      
      //定义反转的数
      reversenum := 0
      //若为回文数，则去除末位依然比反转数大
      for x > reversenum {
          reversenum = reversenum * 10 + x % 10
          x /= 10
      }
      
      //若值的数位为偶数判断是否相等，为奇数则除以10判断是否相等
      return x == reversenum || x == reversenum / 10
  }
  ```

  ```php
  class Solution {
      /**
       * @param Integer $x
       * @return Boolean
       */
      function isPalindrome($x) {
          if($x < 0) return false;
          //将整数转为字符串
          $str = strval($x);
          $len = strlen($str);
          $i = 0;
          while($i < $len/2) {
              if($str[$i] != $str[$len - $i - 1]) {
                  return false;
              }
              $i++;
          }
  
          return true;
      }
  }
  ```

- `最长公共前缀`

  ```go
  func longestCommonPrefix(strs []string) string {
      if len(strs) < 1 {
          return ""
      }
      
      //以第一个字符串进行匹配
      prefix := strs[0]
      for _,str := range strs {
          for strings.Index(str,prefix) != 0 {
              if len(prefix) == 0 {
                  return ""
              }
              //舍掉最后一位
              prefix = prefix[:len(prefix)-1]
          }
      }
      
      return prefix
  }
  ```

  ```php
  class Solution {
      /**
       * @param String[] $strs
       * @return String
       */
      function longestCommonPrefix($strs) {
          if(count($strs) == 1) return array_pop($strs);
          
          public = '';
          $one = array_pop($strs);
          for($i=0;$i<strlen($one);$i++){
              foreach($strs as $item){
                  if($one[$i] <> $item[$i]){
                      return $public;
                  }
              }
              $public .= $one[$i];
          }
          
          return $public;
      }
  }
  ```

### 双指针

- `删除有序数组中的重复项：`有一个升序数组`nums`，请删除重复元素并返回新数组的长度，元素的相对顺序应保持一致（由于在某些语言中不能改变数组的长度，所以必须将结果放在数组`nums`的第一部分。即在删除重复项之后有 `k` 个元素，那么 `nums` 的前 `k` 个元素应该保存最终结果（不要使用额外的空间）

  ```go
  func removeDuplicates(nums []int) int {
      if len(nums) == 0 {
          return 0
      }
      //定义快慢指针
      slow := 1	//表示返回的数目
      for fast := 1; fast < len(nums); fast++ {
          if nums[fast] != nums[fast-1] {
              nums[slow] = nums[fast]
              slow++
          }
      }
      
      return slow
  }
  ```

  ```php
  class Solution {
      /**
       * @param Integer[] $nums
       * @return Integer
       */
      function removeDuplicates(&$nums) {
          if(count($nums) == 0)   return 0;
          $slow = 1;
          for($fast = 1; $fast < count($nums); $fast++) {
              if($nums[$fast] != $nums[$fast - 1]) {
                  $nums[$slow] = $nums[$fast];
                  $slow++;
              }
          }
  
          return $slow;
      }
  }
  ```

- `移除元素：`给定一个数组`nums`和一个值`val`，原地移除所有值为`val`的元素，并返回移除后数组的新长度

  ```go
  func removeElement(nums []int,val int) int {
      if len(nums) == 0 {
          return 0
      }
      slow := 0
      for fast := 0; fast < len(nums); fast++ {
          if nums[fast] != val {
              nums[slow] = nums[fast]
              slow++
          }
      }
      
      return slow
  }
  ```

  ```php
  class Solution {
      /**
       * @param Integer[] $nums
       * @param Integer $val
       * @return Integer
       */
      function removeElement(&$nums, $val) {
          if(count($nums) == 0) return 0;
          $slow = 0;
          for($fast = 0; $fast < count($nums); $fast++) {
              if($nums[$fast] != $val){
                  $nums[$slow] = $nums[$fast];
                  $slow++;
              }
          }
  
          return $slow;
      }
  }
  ```

### 栈

- `有效的括号：`给定一个只包括括号的字符串`s`，判断字符串是否有效

  ```
  示例1：
  	输入：s = "()"
  	输出：true
  示例2：
  	输入：s = "(]"
  	输出：false
  ```

  ```go
  func isValid(s string) bool {
      if len(s) % 2 == 1 {
          return false
      }
  
      //定义包含括号的栈对照字符串
      pairs := map[byte]byte{
          ')':'(',
          ']':'[',
          '}':'{',
      }
      //定义存储左括号的切片栈
      stack := []byte{}
      
      for i := 0; i < len(s); i++ {
          //若匹配到右括号
          if pairs[s[i]] > 0 {
              //栈内无对应的左括号
              if len(stack) == 0 || stack[len(stack)-1] != pairs[s[i]] {
                  return false
              }
              stack = stack[:len(stack)-1]
          } else {
              stack = append(stack,s[i])
          }
      } 
  
      return len(stack) == 0
  }
  ```

  ```php
  class Solution {
      /**
       * @param String $s
       * @return Boolean
       */
      function isValid($s) {
          $stack = new Splstack();
  
          for($i=0;$i<strlen($s);$i++){
              if($s[$i] == '(')   $stack->push(')');
              elseif($s[$i] == '[')   $stack->push(']');
              elseif($s[$i] == '{')   $stack->push('}');
              elseif($stack->isEmpty() || $stack->top() != $s[$i])    return false;
              else    $stack->pop();
          }
  
          return $stack->isEmpty();
      }
  }
  ```

### 链表

- `合并两个有序链表：`将两个升序链表合并为一个新的升序链表并返回

  ```
  输入：l1 = [1,2,4],l2 = [1,3,4]
  输出：[1,1,2,3,4,4]
  ```

  ```go
  /**
   * Definition for singly-linked list.
   * type ListNode struct {
   *     Val int
   *     Next *ListNode
   * }
   */
  func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
      //定义一个合并链表
      pumyHead := &ListNode{}
      //定义头指针
      p := pumyHead
      for list1 != nil && list2 != nil {
          if list1.Val >= list2.Val {
              p.Next = list2
              list2 = list2.Next
          } else {
              p.Next = list1
              list1 = list1.Next
          }
          p = p.Next
      }
      if list1 != nil {
          p.Next = list1
      }
      if list2 != nil {
          p.Next = list2
      }
  
      return pumyHead.Next
  }
  ```

  ```php
  /**
   * Definition for a singly-linked list.
   * class ListNode {
   *     public $val = 0;
   *     public $next = null;
   *     function __construct($val = 0, $next = null) {
   *         $this->val = $val;
   *         $this->next = $next;
   *     }
   * }
   */
  class Solution {
      /**
       * @param ListNode $list1
       * @param ListNode $list2
       * @return ListNode
       */
      function mergeTwoLists($list1, $list2) {
          if($list1 == null) return $list2;
          if($list2 == null) return $list1;
          if($list1->val <= $list2->val){
              $list1->next = $this->mergeTwoLists($list1->next,$list2);
              return $list1;
          } else {
              $list2->next = $this->mergeTwoLists($list1,$list2->next);
              return $list2;
          }
      }
  }
  ```

  
