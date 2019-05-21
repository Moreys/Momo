# 1903LeetCode

## 三月

### 001、两数之和

```c++
/*
 * 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。
示例:
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
 */
#include<vector>
#include<list>
#include <unordered_map>
using namespace std;
class Solution {
public:
	vector<int> twoSum(vector<int>& nums, int target) {
				
		//1两个for循环依次对比数组 
		//时间间复杂度：O(n^2)
		//空间复杂度：O(1)
		for(int i = 0;i < nums.size();++i)
		{			
			for(int j = i +1;j < nums.size();++j)
			{
			if((nums[i] + nums[j]) == target)
			{
				return { i,j };
			}
			}
		}
		//2 使用哈希表
		vector<int>vec;
		std::unordered_map<int, int>nu_map;
		for(int i = nums.size() -1;i >= 0;nu_map[nums[i]] = i,i--)
		{
			if(nu_map.find(target - nums[i]) == nu_map.end())
				continue;
			vec.push_back(i);
			vec.push_back(nu_map[target - nums[i]]);
			return vec;
		}
		return  vec;
	}
};

```

### 026删除数组重复数字

### 027、移除数组

```c++
/*
 * 给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例 1:

给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。

你不需要考虑数组中超出新长度后面的元素。
示例 2:

给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

注意这五个元素可为任意顺序。

你不需要考虑数组中超出新长度后面的元素
 */
#include<iostream>
#include<vector>
#include<list>
#include<string>
using namespace std;
int removeElement(vector<int>& nums, int val) 
{
	int i = 0;
	for (int j = 0; j < nums.size(); j++)
	{
		if (nums[j] != val)
		{
			nums[i] = nums[j];
			i++;
		}
	}
	return i;
}
int main()
{
	std::vector<int>vec = { 1,2,3,4,4,5 };
	int i = 4;
	std::cout << removeElement(vec,i);
	system("pause");
	return 0;
}
```

### 028、实现strStr

```c++
/*
 * 实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:

输入: haystack = "hello", needle = "ll"
输出: 2
示例 2:

输入: haystack = "aaaaa", needle = "bba"
输出: -1
 */
#include<iostream>
#include<vector>
#include<list>
#include<string>
using namespace std;
int strStr(string haystack, string needle) {
	// 如果needle是空字符串，则返回0
	if (needle.empty()) {
		return 0;
	}
	// 获得needle的长度，作为滑动窗口大小
	size_t winLen = needle.size();
	if (haystack.empty() || needle.size() > haystack.size()) {
		return -1;
	}
	// 初始化一个空字符串，作为比较的变量
	for (int i{ 0 }; i <= haystack.size() - winLen; i++) {
		string temp(haystack, i, winLen);
		if (temp == needle) {
			return i;
		}
	}
	return -1;
}
int main()
{
	system("pause");
	return 0;
}
```

