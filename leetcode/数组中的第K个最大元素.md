```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        nums = quSort(nums, 0, nums.length - 1);
        return nums[nums.length - k];
    }


    public static int[] quSort(int[] arr, int start, int end) {
        if (start >= end) {
            return arr;
        }
        int temp = arr[start];
        int i = start;
        int j = end;
        while(i != j) {
            while(arr[j] >= temp && i < j) {
                j--;
            }
            while(arr[i] <= temp && i < j) {
                i++;
            }
            if (i < j) {
                int t = arr[i];
                arr[i] = arr[j];
                arr[j] = t;
            }
        }
        arr[start] = arr[i];
        arr[i] = temp;
        quSort(arr, start, i);
        quSort(arr, i + 1, end);
        return arr;
    }
}
```

快排。