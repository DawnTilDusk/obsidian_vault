- [[#目的：用于离线寻找`rank(k)`元素|目的：用于离线寻找`rank(k)`元素]]
- [[#核心步骤|核心步骤]]
- [[#代码实现|代码实现]]
	- [[#代码实现#1. 最直接：维护两个`vector<int> l, r`|1. 最直接：维护两个`vector<int> l, r`]]
	- [[#代码实现#2. 使用双指针，然后实时`swap()`|2. 使用双指针，然后实时`swap()`]]
- [[#拓展：其他情况下的求`rank(k)`的算法|拓展：其他情况下的求`rank(k)`的算法]]
	- [[#拓展：其他情况下的求`rank(k)`的算法#1. 如果数据是静态的：快速选择算法 (Quick Select)|1. 如果数据是静态的：快速选择算法 (Quick Select)]]
	- [[#拓展：其他情况下的求`rank(k)`的算法#2. 如果数据是流式的：用“小根堆” (Min-Heap)|2. 如果数据是流式的：用“小根堆” (Min-Heap)]]
	- [[#拓展：其他情况下的求`rank(k)`的算法#3. 各种方法的对比|3. 各种方法的对比]]

## 目的：用于离线寻找`rank(k)`元素

**快速选择（Quick Select）** 的核心逻辑和 **快速排序（Quick Sort）** 几乎一模一样，但它更“偷懒”：快排要处理左右两边，而快速选择**只处理一边**。

它的目标是：找到数组排序后 index 为 `target` 的那个数。

---
## 核心步骤

假设我们要找第 $k$ 大的数，对应的下标是 $target = n - k$（如果是找第 $k$ 小，下标就是 $k-1$）。

1. **选择基准值 (Pivot)：** 随机选一个数（通常选中间或末尾）。
2. **分区 (Partition)：** 把比 Pivot 小的换到左边，比 Pivot 大的换到右边。
3. **判断位置：** 此时，Pivot 所在的位置 $p$ 就是它在最终有序数组中的**准确位置**。
    - 如果 $p == target$：恭喜，找到了，直接返回！
    - 如果 $p > target$：说明目标在左半部分，递归左边。
    - 如果 $p < target$：说明目标在右半部分，递归右边。

## 代码实现

### 1. 最直接：维护两个`vector<int> l, r` 

>[!WARNING]
>注意这里一旦这样操作，**空间复杂度就会非常大**，
>而且实践证明，**这样pushback的速度极其缓慢**，仅作为一种参考方法
>**实战中不倡导这样写代码！！**

```cpp
class Solution {
public:
    int qs(vector<int> &nums, int k){
        int piv = nums.back(), rep = 1;
        vector<int> l, r;
        for(int i=0; i<nums.size()-1; i++){
            if(nums[i] < piv) l.push_back(nums[i]);
            else if(nums[i] > piv) r.push_back(nums[i]);
            else rep++;
        }
        if(r.size()+1 <= k && k <= r.size()+rep) return piv;
        else if(k > r.size() + rep) return qs(l, k - r.size() - rep);
        else return qs(r, k);
    }

    int findKthLargest(vector<int>& nums, int k) {
        return qs(nums, k);
    }
};
```
### 2. 使用双指针，然后实时`swap()`
```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        // 目标：第 k 大即升序排序后的下标 n - k
        return quickSelect(nums, 0, nums.size() - 1, nums.size() - k);
    }

    int quickSelect(vector<int>& nums, int left, int right, int target) {
        if (left == right) return nums[left];

        // 1. 随机选取 pivot，防止最坏情况 O(n^2)
        int pivotIndex = left + rand() % (right - left + 1);
        int pivot = nums[pivotIndex];
        
        // 2. 三路划分 (Dutch National Flag logic)
        // 将数组分为：[小于pivot] [等于pivot] [大于pivot]
        int lt = left;      // 指向第一个等于 pivot 的位置
        int i = left;       // 当前遍历指针
        int gt = right;     // 指向最后一个等于 pivot 的位置
        
        while (i <= gt) {
            if (nums[i] < pivot) swap(nums[i++], nums[lt++]);
            else if (nums[i] > pivot) swap(nums[i], nums[gt--]);
            else i++;
        }

        // 此时：[left...lt-1] < pivot, [lt...gt] == pivot, [gt+1...right] > pivot
        if (target >= lt && target <= gt) return nums[lt];
        if (target < lt) return quickSelect(nums, left, lt - 1, target);
        return quickSelect(nums, gt + 1, right, target);
    }
};
```

## 拓展：其他情况下的求`rank(k)`的算法

### 1. 如果数据是静态的：快速选择算法 (Quick Select)

如果你有一个固定的数组，只想找出其中第 $k$ 大的数，最快的办法不是排序，也不是用堆，而是**快速选择**。

- **原理：** 借用“快速排序”的分治思想。每次选一个基准值（Pivot），把数组分成“比它大”和“比它小”的两部分。
- **为什么快：** 快排要递归处理左右两边，而快速选择只需递归其中一边。
- **时间复杂度：** 平均 **$O(n)$**。

---
### 2. 如果数据是流式的：用“小根堆” (Min-Heap)

这听起来可能有点反直觉：**找“第 k 大”，为什么要用“小根堆”？**
想象一个容器，它的容量只有 $k$。我们希望这个容器里装的是目前见过的“前 $k$ 个最大的数”。

- **逻辑：**
    1. 维持一个大小为 $k$ 的**小根堆**。
    2. 堆顶是这个容器里的“守门员”（最小值）。
    3. 每进来一个新数：
        - 如果比堆顶还小，直接扔掉（它不可能是前 $k$ 大）。
        - 如果比堆顶大，就把堆顶踢走，让新数进来，然后重新调整堆。
- **结果：** 遍历完所有数据后，堆顶那个“守门员”就是第 $k$ 大的数。
- **时间复杂度：** $O(n \log k)$。

---
### 3. 各种方法的对比

|**方法**|**时间复杂度**|**适用场景**|
|---|---|---|
|**直接排序**|$O(n \log n)$|数据量极小，图省事|
|**快速选择**|**$O(n)$**|静态数组，追求极致速度|
|**小根堆**|$O(n \log k)$|**大数据量**或**流式数据**（内存放不下所有数据）|
|**二叉搜索树**|$O(\log n)$|频繁插入且频繁查询第 $k$ 大|