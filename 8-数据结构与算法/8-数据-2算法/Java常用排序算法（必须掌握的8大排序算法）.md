版权信息
原文链接： [Java常用排序算法/程序员必须掌握的8大排序算法](http://blog.csdn.net/qy1387/article/details/7752973)

---
本文由网络资料整理而来，如有问题，欢迎指正！

分类：

1）插入排序（直接插入排序、希尔排序）
2）交换排序（冒泡排序、快速排序）
3）选择排序（直接选择排序、堆排序）
4）归并排序
5）分配排序（基数排序）
所需辅助空间最多：归并排序
所需辅助空间最少：堆排序
平均速度最快：快速排序 

不稳定：快速排序，希尔排序，堆排序。

**![](http://img.my.csdn.net/uploads/201209/07/1347008904_9606.jpg)** 

~~~java
// 排序原始数据
private static final int[] NUMBERS =
{49, 38, 65, 97, 76, 13, 27, 78, 34, 12, 64, 5, 4, 62, 99, 98, 54, 56, 17, 18, 23, 34, 15, 35, 25, 53, 51};
~~~
** 1\. 直接插入排序**

基本思想：在要排序的一组数中，假设前面(n-1)[n>=2] 个数已经是排

好顺序的，现在要把第n个数插到前面的有序数中，使得这n个数

也是排好顺序的。如此反复循环，直到全部排好顺序。

![](http://img.my.csdn.net/uploads/201209/07/1347008997_4015.jpg)
~~~java
 public static void insertSort(int[] array) {
     for (int i = 1; i < array.length; i++) {
         int temp = array[i];
         int j = i - 1;
         for (; j >= 0 && array[j] > temp; j--) {
             //将大于temp的值整体后移一个单位
             array[j + 1] = array[j];
         }
         array[j + 1] = temp;
     }
     System.out.println(Arrays.toString(array) + " insertSort");
 }
~~~

**2**. **希尔排序**

希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。希尔排序是非稳定排序算法。
希尔排序是基于插入排序的以下两点性质而提出改进方法的：
插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位。
先取一个正整数d1 < n, 把所有相隔d1的记录放一组，每个组内进行直接插入排序；然后d2 < d1，重复上述分组和排序操作；直至di = 1，即所有记录放进一个组中排序为止。

![](http://img.blog.csdn.net/20170627134544822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXkxMzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
~~~java
public static void shellSort(int[] array) {
    int i;
    int j;
    int temp;
    int gap = 1;
    int len = array.length;
    while (gap < len / 3) { gap = gap * 3 + 1; }
    for (; gap > 0; gap /= 3) {
        for (i = gap; i < len; i++) {
            temp = array[i];
            for (j = i - gap; j >= 0 && array[j] > temp; j -= gap) {
                array[j + gap] = array[j];
            }
            array[j + gap] = temp;
        }
    }
    System.out.println(Arrays.toString(array) + " shellSort");
}
~~~
 

**3**. **简单选择排序**

基本思想：在要排序的一组数中，选出最小的一个数与第一个位置的数交换；

然后在剩下的数当中再找最小的与第二个位置的数交换，如此循环到倒数第二个数和最后一个数比较为止。

![](http://img.my.csdn.net/uploads/201209/07/1347009182_8908.jpg)
~~~java
public static void selectSort(int[] array) {
    int position = 0;
    for (int i = 0; i < array.length; i++) {
        int j = i + 1;
        position = i;
        int temp = array[i];
        for (; j < array.length; j++) {
            if (array[j] < temp) {
                temp = array[j];
                position = j;
            }
        }
        array[position] = array[i];
        array[i] = temp;
    }
    System.out.println(Arrays.toString(array) + " selectSort");
}
~~~
 

**4**. **堆排序**

基本思想：堆排序是一种树形选择排序，是对直接选择排序的有效改进。

堆的定义如下：具有n个元素的序列（h1,h2,...,hn),当且仅当满足（hi>=h2i,hi>=2i+1）或（hi<=h2i,hi<=2i+1）(i=1,2,...,n/2)时称之为堆。在这里只讨论满足前者条件的堆。由堆的定义可以看出，堆顶元素（即第一个元素）必为最大项（大顶堆）。完全二叉树可以很直观地表示堆的结构。堆顶为根，其它为左子树、右子树。初始时把要排序的数的序列看作是一棵顺序存储的二叉树，调整它们的存储序，使之成为一个堆，这时堆的根节点的数最大。然后将根节点与堆的最后一个节点交换。然后对前面(n-1)个数重新调整使之成为堆。依此类推，直到只有两个节点的堆，并对它们作交换，最后得到有n个节点的有序序列。从算法描述来看，堆排序需要两个过程，一是建立堆，二是堆顶与堆的最后一个元素交换位置。所以堆排序有两个函数组成。一是建堆的渗透函数，二是反复调用渗透函数实现排序的函数。

建堆：

![](http://img.my.csdn.net/uploads/201209/07/1347009276_4525.jpg)

交换，从堆中踢出最大数

![](http://img.my.csdn.net/uploads/201209/07/1347009298_9638.jpg)

剩余结点再建堆，再交换踢出最大数

![](http://img.my.csdn.net/uploads/201209/07/1347009312_2298.jpg)

依次类推：最后堆中剩余的最后两个结点交换，踢出一个，排序完成。
~~~java
public static void heapSort(int[] array) {
    /*
     *  第一步：将数组堆化
     *  beginIndex = 第一个非叶子节点。
     *  从第一个非叶子节点开始即可。无需从最后一个叶子节点开始。
     *  叶子节点可以看作已符合堆要求的节点，根节点就是它自己且自己以下值为最大。
     */
    int len = array.length - 1;
    int beginIndex = (len - 1) >> 1;
    for (int i = beginIndex; i >= 0; i--) {
        maxHeapify(i, len, array);
    }
    /*
     * 第二步：对堆化数据排序
     * 每次都是移出最顶层的根节点A[0]，与最尾部节点位置调换，同时遍历长度 - 1。
     * 然后从新整理被换到根节点的末尾元素，使其符合堆的特性。
     * 直至未排序的堆长度为 0。
     */
    for (int i = len; i > 0; i--) {
        swap(0, i, array);
        maxHeapify(0, i - 1, array);
    }
    System.out.println(Arrays.toString(array) + " heapSort");
}
private static void swap(int i, int j, int[] arr) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
/**
 * 调整索引为 index 处的数据，使其符合堆的特性。
 *
 * @param index 需要堆化处理的数据的索引
 * @param len   未排序的堆（数组）的长度
 */
private static void maxHeapify(int index, int len, int[] arr) {
    int li = (index << 1) + 1; // 左子节点索引
    int ri = li + 1;           // 右子节点索引
    int cMax = li;             // 子节点值最大索引，默认左子节点。
    if (li > len) {
        return;       // 左子节点索引超出计算范围，直接返回。
    }
    if (ri  arr[li]) // 先判断左右子节点，哪个较大。
    { cMax = ri; }
    if (arr[cMax] > arr[index]) {
        swap(cMax, index, arr);      // 如果父节点被子节点调换，
        maxHeapify(cMax, len, arr);  // 则需要继续判断换下后的父节点是否符合堆的特性。
    }
}
~~~
 

**5**. **冒泡排序**

基本思想：在要排序的一组数中，对当前还未排好序的范围内的全部数，自上而下对相邻的两个数依次进行比较和调整，让较大的数往下沉，较小的往上冒。即：每当两相邻的数比较后发现它们的排序与排序要求相反时，就将它们互换。

![](http://img.my.csdn.net/uploads/201209/07/1347009396_8149.jpg)
~~~java
public static void bubbleSort(int[] array) {
    int temp = 0;
    for (int i = 0; i < array.length - 1; i++) {
        for (int j = 0; j < array.length - 1 - i; j++) {
            if (array[j] > array[j + 1]) {
                temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
            }
        }
    }
    System.out.println(Arrays.toString(array) + " bubbleSort");
}
~~~
**6**. **快速排序**

基本思想：选择一个基准元素,通常选择第一个元素或者最后一个元素,通过一趟扫描，将待排序列分成两部分,一部分比基准元素小,一部分大于等于基准元素,此时基准元素在其排好序后的正确位置,然后再用同样的方法递归地排序划分的两部分。

![](http://img.my.csdn.net/uploads/201209/07/1347009479_6587.jpg)
~~~java
public static void quickSort(int[] array) {
    _quickSort(array, 0, array.length - 1);
    System.out.println(Arrays.toString(array) + " quickSort");
}

private static int getMiddle(int[] list, int low, int high) {
    int tmp = list[low];    //数组的第一个作为中轴
    while (low < high) {
         while (tmp < list[high]) {
                high--;
            }

        list[low] = list[high];   //比中轴小的记录移到低端
        while (low < high && list[low] <= tmp) {
            low++;
        }

        list[high] = list[low];   //比中轴大的记录移到高端
    }
    list[low] = tmp;              //中轴记录到尾
    return low;                  //返回中轴的位置
}

private static void _quickSort(int[] list, int low, int high) {
    if (low < high) {
        int middle = getMiddle(list, low, high);  //将list数组进行一分为二
        _quickSort(list, low, middle - 1);      //对低字表进行递归排序
        _quickSort(list, middle + 1, high);      //对高字表进行递归排序
    }
}
~~~
**7、归并排序**

基本排序：归并（Merge）排序法是将两个（或两个以上）有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列是有序的。然后再把有序子序列合并为整体有序序列。

![](http://img.my.csdn.net/uploads/201209/07/1347009541_6721.jpg)
~~~java
public static void mergingSort(int[] array) {
    sort(array, 0, array.length - 1);
    System.out.println(Arrays.toString(array) + " mergingSort");
}

private static void sort(int[] data, int left, int right) {
    if (left < right) {
        //找出中间索引
        int center = (left + right) / 2;
        //对左边数组进行递归
        sort(data, left, center);
        //对右边数组进行递归
        sort(data, center + 1, right);
        //合并
        merge(data, left, center, right);
    }
}

private static void merge(int[] data, int left, int center, int right) {
    int[] tmpArr = new int[data.length];
    int mid = center + 1;
    //third记录中间数组的索引
    int third = left;
    int tmp = left;
    while (left <= center && mid <= right) {
        //从两个数组中取出最小的放入中间数组
        if (data[left] <= data[mid]) {
            tmpArr[third++] = data[left++];
        } else {
            tmpArr[third++] = data[mid++];
        }
    }

    //剩余部分依次放入中间数组
    while (mid <= right) {
        tmpArr[third++] = data[mid++];
    }

    while (left <= center) {
        tmpArr[third++] = data[left++];
    }

    //将中间数组中的内容复制回原数组
    while (tmp <= right) {
        data[tmp] = tmpArr[tmp++];
    }
}
~~~
**8、基数排序**

基本思想：将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后,数列就变成一个有序序列。

![](http://img.my.csdn.net/uploads/201209/07/1347009583_9101.jpg)
~~~java
public static void radixSort(int[] array) {
    //首先确定排序的趟数;
    int max = array[0];
    for (int i = 1; i < array.length; i++) {
        if (array[i] > max) {
            max = array[i];
        }
    }
    int time = 0;
    //判断位数;
    while (max > 0) {
        max /= 10;
        time++;
    }

    //建立10个队列;
    ArrayList> queue = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        ArrayList queue1 = new ArrayList<>();
        queue.add(queue1);
    }

    //进行time次分配和收集;
    for (int i = 0; i < time; i++) {
        //分配数组元素;
        for (int anArray : array) {
            //得到数字的第time+1位数;
            int x = anArray % (int)Math.pow(10, i + 1) / (int)Math.pow(10, i);
            ArrayList queue2 = queue.get(x);
            queue2.add(anArray);
            queue.set(x, queue2);
        }
        int count = 0;//元素计数器;
        //收集队列元素;
        for (int k = 0; k < 10; k++) {
            while (queue.get(k).size() > 0) {
                ArrayList queue3 = queue.get(k);
                array[count] = queue3.get(0);
                queue3.remove(0);
                count++;
            }
        }
    }
    System.out.println(Arrays.toString(array) + " radixSort");
}
~~~
**结果**

**![](http://img.blog.csdn.net/20170627133629825?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXkxMzg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)** 

