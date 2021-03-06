###红黑树
- 树状结构，RBTree实现。
    - 插入删除不需要数据复制。

    - 操作复杂度仅跟树高有关。

- RBTree本身也是二叉排序树的一种，key值有序，且唯一。
    - 必须保证key可排序。

基于红黑树实现的map结构（实际上是map, set, multimap，multiset底层均是红黑树），不仅增删数据时不需要移动数据，其所有操作都可以在O(logn)时间范围内完成。另外，基于红黑树的map在通过迭代器遍历时，得到的是key按序排列后的结果，这点特性在很多操作中非常方便。红黑树一些基本概念是需要掌握的。

1. 它是二叉排序树（继承二叉排序树特显）：
    - 若左子树不空，则左子树上所有结点的值均小于或等于它的根结点的值。

    - 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值。

    - 左、右子树也分别为二叉排序树。

2. 它满足如下几点要求：
    - 树中所有节点非红即黑。

    - 根节点必为黑节点。

    - 红节点的子节点必为黑（黑节点子节点可为黑）。

    - 从根到NULL的任何路径上黑结点数相同。

3. 查找时间一定可以控制在O(logn)。

4. 红黑树的节点定义如下：
    ```C++
    enum Color {
        RED = 0,
        BLACK = 1
    };
    struct RBTreeNode {
        struct RBTreeNode*left, *right, *parent;
        int key;
        int data;
        Color color;
    };
    ```

所以对红黑树的操作需要满足两点：1.满足二叉排序树的要求；2.满足红黑树自身要求。通常在找到节点通过和根节点比较找到插入位置之后，还需要结合红黑树自身限制条件对子树进行左旋和右旋。

相比于AVL树，红黑树平衡性要稍微差一些，不过创建红黑树时所需的旋转操作也会少很多。相比于最简单的BST，BST最差情况下查找的时间复杂度会上升至O(n)，而红黑树最坏情况下查找效率依旧是O(logn)。所以说红黑树之所以能够在STL及Linux内核中被广泛应用就是因为其折中了两种方案，既减少了树高，又减少了建树时旋转的次数。

从红黑树的定义来看，红黑树从根到NULL的每条路径拥有相同的黑节点数（假设为n），所以最短的路径长度为n（全为黑节点情况）。因为红节点不能连续出现，所以路径最长的情况就是插入最多的红色节点，在黑节点数一致的情况下，最可观的情况就是黑红黑红排列......最长路径不会大于2n，这里路径长就是树高。

###STL内存配置器

　　STL内存分配分为一级分配器和二级分配器，一级分配器就是采用malloc分配内存，二级分配器采用内存池。

　　二级分配器设计的非常巧妙，分别给8k，16k,..., 128k等比较小的内存片都维持一个空闲链表，每个链表的头节点由一个数组来维护。需要分配内存时从合适大小的链表中取一块下来。假设需要分配一块10K的内存，那么就找到最小的大于等于10k的块，也就是16K，从16K的空闲链表里取出一个用于分配。释放该块内存时，将内存节点归还给链表。
如果要分配的内存大于128K则直接调用一级分配器。
为了节省维持链表的开销，采用了一个union结构体，分配器使用union里的next指针来指向下一个节点，而用户则使用union的空指针来表示该节点的地址。


###排序算法
　　STL中的sort()，在数据量大时，采用quicksort，分段递归排序；一旦分段后的数量小于某个门限值，改用Insertion sort，避免quicksort深度递归带来的过大的额外负担，如果递归层次过深，还会改用heapsort。

　　sort采用的是成熟的"快速排序算法"(目前大部分STL版本已经不是采用简单的快速排序，而是结合内插排序算法) 可以保证很好的平均性能、复杂度，stable_sort采用的是"归并排序"，分派足够内存时，其算法复杂度为n*log(n), 否则其复杂度为n*log(n)*log(n)，其优点是会相等元素的顺序不会改变。


### \++iterator 和 iterator++效率问题？

两种方式iterator遍历的次数是相同的，但在STL中效率不同，前\++返回引用，后\++返回一个临时对象，因为iterator是类模板，使用 it\++这种形式要返回一个无用的临时对象，it\++是重载，所以无法对其进行优化，所以每遍历一个元素，你就创建并销毁了一个无用的临时对象。
除了特殊需要和对内置类型外使用++it来进行元素遍历的。

###STL中的vector：增减元素对迭代器的影响？

这个问题主要是针对连续内存容器和非连续内存容器。

a、对于连续内存容器，如vector、deque等，增减元素均会使得当前之后的所有迭代器失效。因此，以删除元素为例：由于erase()总是返回指向被删除元素的下一个元素的有效迭代器，因此，可以利用该连续内存容器的成员erase()函数的返回值。常见的编程写法为：

```cpp

 for(auto iter = myvec.begin(); iter != myvec.end())
 {
    iter = myvec.erase(iter);  
 }
```
还有两种极端的情况是：

(1)vector插入元素时位置过于靠前，导致需要后移的元素太多，因此vector增加元素建议使用push_back而非insert；

(2)当增加元素后整个vector的大小超过了预设，这时会导致vector重新分分配内存，效率极低。因此习惯的编程方法为：在声明了一个vector后，立即调用reserve函数，令vector可以动态扩容。通常vector是按照之前大小的2倍来增长的。

b、对list 插入、移除、结合等操作，除了指向被移除元素的迭代器会失效，不会造成其他原有迭代器失效。

b、对于非连续内存容器，如set、map等。增减元素只会使得当前迭代器无效。仍以删除元素为例，由于删除元素后，erase()返回的迭代器将是无效的迭代器。因此，需要在调用erase()之前，就使得迭代器指向删除元素的下一个元素。常见的编程写法为：

```cpp
 for(autoiter = myset.begin(); iter != myset.end()) 

 {
    myset.erase(iter++);  //使用一个后置自增就OK了  
 }
```