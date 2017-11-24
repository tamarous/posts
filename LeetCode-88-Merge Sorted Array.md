# LeetCode-88-Merge Sorted Array
这道题也属于合并两个容器这个类型的题目。之前我做过一道合并两个有序链表的题目，那道题的要求是用一个新的链表来存储合并后的链表，因此思路就是很清晰的：

用两个指针分别指向原来两个链表的头节点，新建一个节点，比较指针所指节点的数值的大小，然后把新建的节点的数值赋成数值较小或者较大的节点的数值，然后移动指针的位置。当一个链表的值全部遍历过后，就把另一个链表的值直接拷贝进新的链表中，在这个过程中要调整新建链表节点的 next 指针。

相比起来，本题要显得难一点，因为本题要求的是原地赋值，也就是不能开辟新的空间。

代码如下：

    void merge(int *nums1, int m, int *nums2, int n) {
        if (nums1 == NULL || nums2 == NULL) {
            return;
        }
        int i = m - 1;
        int j = n - 1;
        int k = m + n - 1;
        while(i >= 0 && j >= 0) {
            if (nums1[i] > nums2[j]) {
                nums1[k--] = nums1[i--];
            } else {
                nums1[k--] = nums2[j--];
            }
        }
        while(j >= 0) {
            nums1[k--] = nums2[j--];
        }
    }

