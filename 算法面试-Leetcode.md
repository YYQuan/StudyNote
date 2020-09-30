### 排序



排序算法的效率的指标

- 时间复杂度

- 内存消耗

- 是否稳定

  稳定是指 算法执行前后 各个元素的前后顺序是否发生变化

#### Bubble Sort 

​	

```java
protect  void bubbleSort(int[] ints){
	for(int i = 0 ; i< ints.length-1 ;i++){
		// 该轮循环中 没有数据交换就说明 已经有序了
        boolean isEnd = true;
		for(int j = 0 ; 
            j <ints.length-i-1;j++){
     		int pre = ints[j]       ;
       		int next = ints[j+1]    ;
            if(pre>next){
                isEnd = false;
                ints[j] = next;
                ints[j+1] = pre;
            }
        }
        if(isEnd){
            break;
        }
    }
}
```



练习

```java

public static void sort(Comparable[] arr){
    if(arr == null ) return ;
    int n = arr.length;
    for(int i = 0;i<n;i++){
        boolean isEnd = true;
        for(int j = 0 ; j< n-i-1 ;j++){
            if(arr[j].compareTo(arr[j+1])>0){
				isEnd = false;
                swap(arr,j,j+1)
            }
        }
        // 如果没有交换过数据 就说明后面都已经是有序的了
        if(isEnd) break;
    }
}

public static void swap(Comparable[] arr,int   indexA ,int indexB){
    if(arr == null) return ;
    int n = arr.length;
    
    if(indexA<0|| indexA>n-1){ return ;}
    if(indexB<0|| indexB>n-1){ return ;}
    
    Comparable tmp = arr[indexA];
    arr[indexA] =arr[idnexB];
    arr[indexB] = tmp;
}
    
```



#### SelectionSort

选出最小的那个元素放到当前最左侧

```java
public static void sort(Comparable[] arr){
    if(arr==null) return ;
    int n = arr.length;
    
    for(int i= 0 ;i<n;i++){
        for(int j = i ;j<n;j++){
			if(arr[i]>arr[j]){
				Comparable  tmp = arr[j];
                arr[j] = arr[i];
                arr[i] = tmp;
            }
        }
	}
}
```



优化

//双路选择排序， 每轮遍历同时取出最大值和最小值

```java
public static void sort(Comparable[] arr){
    
    if(arr == null ) return ;
    int n = arr.length;
    
    for(int i = 0 ,j = n-1;i<=j;i++,j--){
        
        int minIndex = i;
        int maxIndex = j;
        for(int p = i ; p<=j;p++){
					             if(arr[p].compareTo(arr[minIndex])< 0 ){
				minIndex = p;                
            }
            if(arr[p].compareTo(arr[maxIndex]) > 0 ){
				maxIndex = p;                
            }
        }
        
        if(minIndex == j && maxIndex ==i){
            swap(arr,i,j);
        }else if(minIndex == j ){
            swap(arr,j,maxIndex);            
            swap(arr,i,maxIndex);
        }else if(maxIndex == i ){
            swap(arr,i,minIndex);
            swap(arr,j,minIndex);
        }else{
            swap(arr,i,minIndex);
            swap(arr,j,maxIndex);
        }
        
    }
}

public static void swap(Comparable[] arr,int a ,int b){
    if(arr==null) return;
    
    int n = arr.length;
    
    if(a<0||a>=n) return ;
    if(b<0||b>=n) return ;
    
    Comparable tmp = arr[a];
    arr[a] = arr[b];
    arr[b] = tmp ;
}
```



#### InsertionSort

从左 到右， 一个一个的插入左侧 数组，

```java
void  insertionSort(int[] ints){ 
    if(ints==null || ints.length<=1) return ;
    
    for(int i= 1 ; i<ints.length;i++){
		int value = ints[i];
        int j = i -1;
        for(; j>=0;j--){
        	if(ints[j]>value){
                ints[j+1] = ints[j];
            }else{
                ints[j+1] = value;
                break;
            }
        }
    }
}
```

练习

```java
public void insertSort(Comparable[] arr){
    if(arr == null || arr.length <=0 )
        return ;
    
    for(int i = 0 ; i< arr.length ; i++){
        int j = i;
        for( ; j>0 ; j--){
            if(arr[j-1].compareTo(arr[j])>0){
                arr[j] = arr[j-1];
            }
            
        }
        arr[j]= arr[i];
        
    }
    
    return ;
}
```



#### MergeSort(归并)

分治
把问题分离成一个个的两个有序数组合并成一个有序的数组

```java
int[] mergeSort(int[] ints){
    if(ints==null||ints.length<=1) return ints;
    
    return mergeSort(ints,0,ints.length-1);
}

int[] mergeSort(int[]  ints,int start ,int end){
    
    int mid = (start+end)/2;
    int[] intsLeft = mergeSort(ints,start,mid);
    int[] intsRight= mergeSort(ints,mid,end);
    return merge(intsLeft,intsRight);
}

int[] merge(int[] intP,int[] intN){
    if(intP==null&&intN==null){
        return null;
    }else if(intP == null){
        return intN;
    }else if(intN == null){
        return intP;
    }
    
    if(intP.length<=0){
        return intN;
    }else if(intN.length<=0){
        return intP;
    }
    
    int[] result = new int[intP.length+intN.length];
    
    for(int i = 0,j=0,index = 0 ;i<intP.length||j<intN.length;){
		int intI  = intP[i];
        int intJ  = intN[j];
        int min = 0 ;

        if(i<intP.length&&j<intN.length){
            if(intI>intJ){
                i++；
                min = intJ;
            }else{
                j++；
                min = intI;
            }
            result[index++] = min;    
       	}else{
            if(i>=intP.length){
                result[index++] = intN[j++];
            }else{
                result[index++] = intP[i++];
            }
        }
    }
    
    
    return result;
}
```



练习

```java
// 对于java的最普通的归并来说，难点有两个 merge的边界
// 以及merge是 应该是copy数组部分成员 ，而不是全部

public static void sort(Comparable[] arr){
    
    if(arr==null) return;
    sort(arr,0,arr.length);
    
}

private static void sort(Comparable[] arr,int  l,int r){

    if(l>r) return ;
    
    int mid = (l+r)/2;
    sort(arr,l,mid);
    sort(arr,mid+1,r);    
    merge(arr,l,mid,r);
}

private static  void merger(Comparable[] arr, int l, int mid ,int r){
    
    Comparable[] tmp = Arrats.copyOfRange(arr,l,r+1);
    for(int n = l,int i =l,int j = mid+1;n<=r;n++){
        if(i>mid){
            arr[n++] = tmp[j-l];
            j++;
        }else if(j>r){
            arr[n++] = tmp[i-l];
            i++;            
        }else{
            if(tmp[i-l].compareTo(tmp[j-l])>0){
	        	arr[n++] = tmp[i-l];
    	       	j++;            
     
            }else{
    	        arr[n++] = tmp[i-l];
	            i++;            

            }
        }
    }
    
    
    
    
}
```

归并排序优化
优化思路：

- 在小规模数据时，改为使用插入排序
- 对于merge是先判断一下left半部分的最后一个元素A和right半部分的第一个元素B的大小，如果A<B，那么这两个部分本身就是已经排好序的了，就可以直接略过这个对于近乎有序的数组是很有效的，但是对于完全无序的数组反而会增加性能开销

```java
public  void mergeSort(Comparable[] arr){
    if(arr == null || arr.length <= 0 )
        return ;
    
    int end = arr.length-1;

    mergeSort(arr,0,end);
	
}

//[start...end]范围的归并
public void mergeSort(COmparable[] arr,int start ,int end ){
    if(start>=end) return ;
    int mid = (start+end)/2;
    mergeSort(arr,start,mid);
    mergeSort(arr,mid+1,end);
    merge(arr,start,mid,end);
}

private void merge(Comparable[] arr,int start , int mid ,int end ){
    Comparable[] aux = Arrays.copyOfRange(arr,start,end+1);
    
    int p = start ;
    int q = mid +1;
	for(int i = start ; i <=end;i++){
        if(p>mid){
            arr[i] = aux[q-start];
            q++;
        }else if(q<=mid){
            arr[i] = aux[p-start];
            p++;
        }else if( aux[p-start].compareTo(aux[q-start])< 0  ){
            arr[i] = aux[p-start];
            p++;
        }else{
            arr[i] = aux[q-start];
            q++;
        }
    }
}
```





#### QuickSort

##### 单路快排

思路：

- 和归并一样，也是分治的思想
- 先选一个基准值A，小于基准值的归到一起，大于基准值的归到一起

```java


public static  void sort(Comparable[] arr){
    if(arr == null) return ;
    sort(arr,0,arr.length);
}

private static void sort(Comparable[] arr,int l ,int r){
    if(l>r) return ;
    
	int index = partition(arr,l,r);
    sort(arr,l,index-1);
    sort(arr,index+1,r);
}

//把 arr[l] 放在逻辑中间位置， 左侧元素都小于arr[l] 右侧元素都大于arr[l]
private  static  int partition(Comparable[] arr,int l , int r){
	
    Comparable base = arr[l];
    
    int j = l;
    for(int i = l+1 ;i<=r;i++){
        if(arr[i].compareTo(base)<0){
			j++;
            swap(arr,i,j);
        }
    }
    swap(arr,l,j);
 	return  j;   
}
```

练习

```java
   public static  void quickSort(Comparable[] arr){

        if(arr == null || arr.length <= 0 ) return ;

        quickSort(arr,0,arr.length-1);

    }


    //处理范围：[start ... end ]
    public static  void quickSort(Comparable[] arr,int start ,int end){

        if(start > end)  return;;

//        Comparable c = arr[0];
        int index =  handle(arr, start ,end);
        quickSort(arr,start,index-1);
        quickSort(arr,index+1,end);

    }


    //处理范围：[start ... end ]
    private static int handle(Comparable[] arr,int start ,int end) {

        Comparable c = arr[start];

        int j = start;
        for(int i  =start ;i<= end;i++){
            if(arr[i].compareTo(c)<0){
                j++;
                swap(arr, i ,j);
            }
        }
        swap(arr,start,j);
        return j;
    }


    private static void swap(Comparable[] arr, int l , int r){
        Comparable tmp = arr[l];
        arr[l] = arr[r];
        arr[r]= tmp ;
    }

```



优化

思路： 

- 基准的选取应该要随机，否则 用上述的算法的话，对于一个完全有序的数组来说，他的事件复杂度会退化成为O(n^2^)
- 规模小的时候 可以改为使用插入排序

```java
private static void partition(Comparable[] arr,int l ,int r){
   	int randomIndex = Math.random.(r-l+1)+l;
    swap(arr,l,randomIndex);
	Comparable base  = arr[l];
    int j = l ;
    for(int i = l+1;i<=r;i++){
    	if(arr[i].compareTo(base)<0){
        	j++;
            swap(arr,i,j);
        }
    }
    
    swap(arr,l,j);
    return j ;
}
```





##### 双路

优化2

改进成 双路快排
双路快排的优化点：
	当数组相同的元素非常多的时候，就会导致快排左右两个分组不平衡， 在极端情况下，就会退化成O(n^2^)的时间复杂度
	双路快排的思路就是要把相同的元素 平均的分配有左右两边



```java
private static  void partition(Comparable[] arr, int l ,int r){
    
    if(l>r) return ;
    
    int r =(int) Math.random()*(l-r+1);
    swap(arr,r,l);
    Comparable base = arr[l];
    
    for(int i = l+1,j = r;i<j;swap(arr,i,j)){
        // 注意数组越界
        // 不能有等于， 等于的话就不能起到平均分配的效果了 双路就没有意义了
        while(i <= r&&arr[i].compareTo(base)<0)
            i++;
        while(j>= l+1&&arr[j].compareTo(base)>0)
            j--;
        
    }
    
    //这里还有一个坑， 注意是返回J ， 因为先变化的是i,结束时i所对应的值 可能是大于base的 也可能是等于base的 如果是大于base的那就不该和 arr[l]交换 ；但是此时J所对应的值一定是小于等于base的，是一定可以和arr[l]交换的，并且交换后仍然满足 快排的约束条件的 所以应该要返回j （也就是后面再动的那个变量）
    swap(arr,l,j);

    return j;
    
}
```

##### 三路快排

优化：
三路快排
过滤掉等于的部分，减小范围

```java
private  static  void sort(Comparable[] arr,int l ,int r){
	
    int random =（(int)Math.random()*(r-l)+l）;
    swap(arr,random ,l);
    
    Comparable base = arr[l];
    
    //在循环的过程中保持
    
    int lt = l; // arr(l..lt] <base  为了保持该特性 所以lt的初始值是l 而不是l+1
    int rt = r+1;// arr[rt...r]>base 为了保持该特性 所以rt的初始值是r+1 而不是r
    int i = l+1; // arr[lt+1 ... i) == base  这里i初始化为 l 也是可以的，就是会多做一个判断
    
    while(i<rt){
        if(arr[i].compareTo(base)<0)	{
			
            swap(arr,i,lt+1);
            i++;
            lt++;
        }else if(arr[i.compareTo(base)>0]){
			swap(arr,i,rt-1);
            i++;
            rt--;
        }else{
            i++;
        }
    }

    

    swap(arr,l,lt);
	//经过了这个swap后， 
    //元素特性改为：
    //[l...lt-1] <0
    //[lt...i] == 0
    //[i+1...rt] > 0 
    
    sort(arr,l,i-1);
    sort(arr,i+1,r);
}

```

三路快排练习	

```java
private static void sort(Comparable[] arr){
    if(arr == null) return ;
    sort(arr ,0,arr.length-1);
}

private static void sort(Comparable[] arr,int l,int r){
    
    int randomInex = ((int)Math.random()*(r-l+1))+l;
    swap（arr,l,randomIndex );
    
    Combarable base = arr[l];
    //arr(l...lt]<base
    int lt = l;
    //arr[rt...r]>base
    int rt = r+1;
    //arr[lt+1...i) == base
    int i = l+1;
    
    while(i<rt){
        if(arr[i].compareTo（）<0)		{
            // 不要简写成，简写后 看不出变量定义的维护过程
            //swap(arr,++i,++lt);
            
            
            //根据 变量的定义 arr[lt +1]肯定是 == base的
            swap(arr,i,lt+1);
			//swap 之后要维护变量的定义
            i++;
            lt++;
        }else         if(arr[i].compareTo(base)>0){
            swap(arr,i,rt-1)；
            rt--;
    }else {
        	i++;
		}
    
}
```





### Heap(堆)

堆

- 一个完全二叉树
- 父节点一定大于子节点
- 由于是一个完全二叉树，所以可以用数组来比较容易的实现。

堆 经常会被用来做优先队列

先从最简单的最大堆开始

注意堆并不是保证整个数组都是有序的
而是保证父节点 一定比子节点大。也就是说兄弟节点之间是不保证有序的。



PS：

```
		4

  2           3

这种情况是允许的。
```





然后提供一个拿最大值的方法， 每次都拿最大值。
等把堆元素按顺序取完，那就是有序的

取出 最大值之后怎么保证 剩余 下数组中顶部还是最大值呢？

取最大值的时候， 栈顶和最后一元素 交换。然后count -1；  接着把 栈顶的元素指定 shiftDown 操作（要和子节点中大节点交换）。

#### ShiftUp & ShiftDown

```java
Comparable[] arr;// 从1 开始存数据
public void shiftUp(int k){
    while(k>1 && arr[k].compareTo(arr[k/2]>0)){
        swap(arr,k,k/2);
        k = k/2;
    }
}
```



```java
Comparable[] arr;//从1开始存数据
public void shiftDown(){
    int count =arr.length;
    int i  = 1;
    // 数组申请内存的时候最多申请一些的 ，不会发生数组越界
    while(2*i < count){
        int j = 2*i;
    	//筛选出 两个子节点中大的那个
        if(j+1<count&&arr[j].compateTo(arr[j+1])<0)    {
            j++;
        }
        
        // 不小于 大的那个子节点 则说明 已经是最大值了，就不需要在下移了
        if(arr[i].compateTo(arr[j])>=0){
            break;
        }
        swap(arr,i,j);
        // 挪到子节点处 继续判断
        i = j ; 
    }
    
}
```



#### 索引堆

为啥有有索引堆？
因为普通最大堆  进来之后， 就很难再找到特定的元素了。 就只能遍历， 或者 一个个把max push 出来找。

比如一个优先队列， 突然想要改变里面某个元素的优先级，那最大堆就很难处理。
针对该痛点 ，才出现了索引堆。



对于索引堆 ，我们把索引和数据分开存储。
用索引的数组来表征 堆。
比较的时候用的是 数据数组的值，存取序号变化的时候用的是索引数组的值。 

排序前后的对比：

![img](https://images2018.cnblogs.com/blog/1307816/201803/1307816-20180315164001854-1182923332.jpg)

![img](https://images2018.cnblogs.com/blog/1307816/201803/1307816-20180315164217392-1979013270.jpg)



insert & getTop

```java
Comparable[] index;//从1开始存数据
Comparable[] data;// 从1开始存数据
public void insert(Comparable d){
    data[count+1]= d;
    index[count+1] = count+1;
    
    count ++;
    
    shiftUp(count+1);
    
}

private void shiftUp(int n ){
    while(n>1){
        int index1 = index[n]; 
        int index2 = index[n/2];
        if(data[index1].compareTo(data[index2]>0)){
            swap(index,n,n/2);
            
            n = n/2;
        }
    }
}

public Comparable getMax(){
    int  index1 = index[1];
    swap(index,1,count);
    swap(data,1,count);
    
    Comparable result = index[count];
    index[count]=null;
    data[count]=null;    
    count--;    
	shiftDown(1);
    return result;
}

private void shiftDown(int n){
 	//  计数是从1 开始的 所以 是<=
    while(2n<=count){
         int j = 2n;
		if(j+1 <count &&data[index[j]].compareTo(data[index[j+1]])<0){
            j++;
        }
         if(data[index[n]].compareTo(data[index[j]])>=0){
			break;
         }
         
         swap(index,n,j);
     	 n = j;
     }
}

```



### 二分搜索树

#### 二分查找法



递归写法

```java

   // 递归写法

    public static int  binarySearchRes(Comparable[] arr,Comparable c){
        if(arr == null || c == null || arr.length<=0)  return -1;
        return binarySearchResActual(arr,c,0,arr.length-1);
    }


    private  static int  binarySearchResActual(Comparable[] arr , Comparable c, int l , int r){
        if(l>r)  return -1;
        int result = -1;
        int mid = l + (r-l)/2;

        int  compareResult = c.compareTo(arr[mid]);
        if(compareResult == 0)
            result =  mid;
        else if(compareResult >0)
            result = binarySearchResActual(arr,c,mid+1,r);
        else
            result = binarySearchResActual(arr,c,l,mid -1);


        return result;
    }
    
```



非递归写法

```java
    public static int binarySearchTest(Comparable[] arr,Comparable c){
        if(arr == null || c ==null ||  arr.length<=0)  return -1;
        int result = -1;


        int l = 0;
        int r = arr.length -1;

        while(l<=r){

            //int mid = (l+r) /2;
            // 避免溢出的写法
            int  mid = l + (r-l)/2;

            int comperaResult = c.compareTo(arr[mid]);
            // 注意这里的 +1 ，-1  没有+1 -1这个的话 可能会出现   mid = l+(r -l)/2 ;  mid == l  的无限循环
            if(comperaResult>0){
                l = mid +1;
            }else if(comperaResult<0){
                r = mid -1;
            }else{
                result = mid;
                break;
            }

        }

        return  result;
    }

```



#### 二分搜索树遍历



##### 深度优先

二分搜索树的遍历，分前中后 序三种。
可以理解为 每一个节点 都要被访问三次。
第一次访问的时候就读取 就是前序
第二次访问的时候就读取 就是中序
第三次访问的时候就读取 就是后序

示意图

每一个节点都有  红绿蓝 三次访问。
如果是叶子节点的话， 那么第二次访问和第三个访问 绿色和蓝色老天两条先都是指向自己的。

![image-20200917175513676](https://i.loli.net/2020/09/22/hOsfqDPbrE9uHwL.png)



```java


//getValue执行的位置就决定了是前序还是中序还是后序了
public void order(Node root){
    
    //前序
    getvalue();
    order(root.left);
    order(root.right);
    
    
    
    order(root.left);
    //中序
    getvalue();
    order(root.right);
    
    order(root.left);
    order(root.right);
    //后序
    getvalue();
}

```



##### 广度优先（层序遍历）

层序遍历的话  需要借助 队列

```java
public void levelOrder(Node root){
 
    Queue queue = new Queue();
    Node currentNode = root;
    queue.add(currentNode);
    while（!queue.isEmpty()){

        //queue.add(queue.remove());
        //remove和poll的区别是当 queue 没有元素后， remove 会throw个exception  而poll是返回 null
        queue.add(queue.poll());
        
        // 获取 node的值
        getValue();
        
        if(currentNode.left!=null)
            queue.add（currentNode.left);
        if(currentNode.right!=null)
            queue.add(currentNode.right);
        
        
    }
}
```



#### 移除最大 最小节点

```java
// remove min
// 最左的节点就是最小的节点 但是要分 该节点是否有 右子树
// 需要有返回值， 以此来改变父节点的左子树的值
// 注意改变 二分搜索树的总数
public Node removeMin(Node node){
    
    if(node.left == null){
    	count--;
        Node result = null;
        result = node.right;
        node.right  = null;
        return result;
        
    }
    
    // 有左子树就继续往左走
    node.left = removeMind(node.left);
    
    return node;
}


// remove max 
//类比 min
public Node removeMax(Node node){
    if(node.right == null){
        Node result = node.left;
        node.left = null;
        count--;
        return result;
    }
    
    node.right = removeMax(node.right);
    return node;
}
```





#### 删除节点

思路： 
分三种情况：

1. 左子树为空，直接替换
2. 右子树为空， 直接替换
3. 左右子树都不为空， 这就就要取右子树的最左节点来替换到被删除节点的位置

```java
// 返回删除后的 根节点
public Node remove(Node node ,Key key){
    
    if(node == null ) return null;
    
    if(node.getKey()>key){
        
        Node left = remove(node.left,key);
        node.left = left;
        return node;
    }else if(node.getKey()<key){
        Node right = remove(node.right,key);
        node.right = right;
        return node;
    }else{
        //
        if(node.left == null){
        	count-- ;
            
            return node.right;
        }
        
        if(node.right == null){
            count--;
            return node.left;
        }
        
        // 左右都不为空的时候 就要取 最小的大于node的元素，也就是其右子树的 最左元素
        //先得到大于Noded的 最小元素
        Node  replaceNode = getMin(node.right);
        Node  replaceNodeRight  = removeMin(node.right);
        replaceNode.left = node.left;
        replaceNode.right = node.replaceNodeRight;
        node.left = null;
        node.right = null;
        return replaceNode;
        
    }
    
    public Node removeMin(Node node){
        if(node.left == null){
            count--;
            Node result = node.right;
            node.right = null;
            return result;
        }
        node.left = removeMin(node.left);
        return node;
    }
    
    public  Node getMin(Node node){
        if(node.left==null)
            return node;
        return getMin(node.left);
        
    }
    
    
    
    
    
    
}
```





#### 二分搜索树的局限性

![image-20200918102619332](https://i.loli.net/2020/09/22/lJsHQaU9Efi8YKX.png)

所以说 二分搜索树  的树的形状不是 固定的， 是由可能退化成链表的。
所以二分搜索树的 O（nlogn）的时间复杂度是一个估算值。
对于极端情况 是由可能退化成O（n^2^)的。

为了避免出现退化成O(n^2^)的情况，后面才出现avl树（完全平衡二叉树）和 （平衡二叉树）红黑树。





### 并查集

并查集的作用：
判断 两个  元素 是否 连接。
比如 一群人里面 抽两个人 ，快速的判断 他们是不是同个某种属性的人。
逻辑是这样的 元素里保存了自己的上级， 如果两个元素有两个相同的上级 ，那么他们就是连接的。

 两个组的人  如何变成连接关系呢？
把一个组的顶级上级 ，变成另一个组的顶级上级的下级即可。



让结构扁平一些，这样可以增加效率

![image-20200918110023154](https://i.loli.net/2020/09/22/pLIxTn32Q5vRVWg.png)

通过路径压缩来使得结构更加扁平

如果结构1的时候 查询了一次 发现大家的顶级上级都是曹公公，那么就可以直接把查询过程中的每一级的上级都改成直接指向曹公公。

从这里开看，那么并查集最主要的几个操作

1. 添加：  添加时 要告诉和谁是联通的

2. 查询顶级上级

3. 增加连通性  ： 把两个原本不连通 的元素连通起来

4. 查询后 进行路径压缩

   



### leetcode题



####  对撞指针

![image-20200924161842146](https://i.loli.net/2020/09/24/aSnO3g6XATHBqoh.png)

 对撞指针	

```java
//题目中 说了可以假设每个输入值对应唯一答案，所以不需要考虑多个解的情况
```





#### 查找表的经典应用 - 无序数组中twoSum   

![image-20200924163244043](https://i.loli.net/2020/09/24/2iLEv8e1DwrKgtR.png)



和167的对撞指针的题目类似 但是数组是无序的，所以不能用对撞指针。

思路：

-  用map 来存储    数组的 元素值以及序号  ；如果是hashMap add 和find 都是 0(1)的事件复杂度

  

注意点：

- 新add进去元素会把之前的给替换掉，但是题目当中只要求有一组解，所以说在把数组元素put到map当中时，只需要在 map当中查 是否有target-nums[i] 的key , 如果有 就可以直接返回来
  如果没有就遍历下一个元素；
  由于只遍历当前元素前的，因此之前相同的元素被替换了也无所谓。

```java
//解答
public  int[]  twoSum( int[] nums ,int target ){
    
    
    HashMap<Integer,Integer> map = new HashMap();
    
    for(int i = 0 ; i< nums.length ;i++){
        if(map.contain(target - nums[i])){
            int[] result = new int[]{map.get(target - nums[i]),i};
            return result;
        }
        map.put(nums[i],i);
    }
    
    throw new IllegalException(" no  solution ");
}
```



思路2：

- 由于题目中只要求求出唯一的一个解。因此可以在最初的时候遍历整个数组，将数组中的每一个数字的索引放在map中，此时map中记录的永远是每一个数组最后出现的位置，
  对于多个相同值的情况，只要遍历的过程当中 相同值的序号不同，那么就是不同位置的值



```java
public int[] twoSum(int[] nums ,int target){
    HashMap<Integer,Integer> map = new HashMap();
    for(int i = 0;i<nums.length ; i++)
        map.put(nums[i],i);
    
    for(int i = 0 ;i<nums.length; i++){
        int tmp =  target - nums[i];
        if(map.contain(tmp)){
            if(map.get(tmp)!= i ){
                int[] result  = new int[]{map.get(tmp),i};
                return result;
            }
        }
    }
    
    throw new IllegalException("no solution");
    
}
```



#### 两个数组的交集

![image-20200924165953215](https://i.loli.net/2020/09/24/NxQzgtc6HlWfGVT.png)

思路： 利用 set 这种 数据结构来做

参考解法：

![image-20200721162637522](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142037.png)



#### 两个数组的交集II

题号：350

![image-20200721164203856](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142043.png)



![image-20200721164244553](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142055.png)



参考解法：

![image-20200721164258927](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142100.png)



#### 四数相加

题号：454

![image-20200722144055298](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142106.png)



思路：

- 如果直接用暴力解法，那么时间复杂度是O（n^4^） 在数量级在500的情况下，性能肯定是不行的。
- 题目是关心有多少组，并不关系具体的序号。
- 整数的范围在 -2^28^  到2^28^, 所以对于 32位机来说 （2^32^  ） 是不需要考虑越界的问题的
- 把两两数组的合包村到查找表当中， 这个就是两个O(n^2^)复杂度的操作了



参考解法：

![image-20200722144614609](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142114.png)



#### 回旋镖的数量

题号：447
![image-20200722150048748](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142129.png)

题意：

输入 n 个点的坐标， 要求 这些坐标点内 随机取三个点，这三个点 中 :第一个点和第二个点之间的距离和 第一个点和第三个点的距离相同.一共有多少组满足条件的三个点？

思路：

- 把点i到所有其他点的距离出现的频次x 记录到查找表当中
  然后对于每一个点 取A~x~^2^    （注意 不是C~x~^2^，而是A~x~^2^,因为相互调换下顺序就是新的一组） 就可以获取这个点有多少组，把全部点各自的组数相加就是全部的数量了
- 只需要得到有多少组 ，不需要每一组都列出来
- 要避免浮点数 误差， map里记录 距离的平方而不是距离



```java
public  int  numberOfBoomerange1(int[][] points){
    int res = 0;
    HashMap<Integer ,Integer> map = new HashMap();
    for( int i = 0 ;i <points.length;i++){
		
        for(int  j  = 0; j<points.length;j++){
            int dis = pointDis(points[i],points[j]);
            if(dis == 0) break;
            if(map.containsKey(dis)){
                map.put(dis,1+map.get(dis));
            }else{
                map.put(dis,1);
            }
        }
    }
    
    //A ~n~ ^2^
    for(int dis: map){
		res +=  map.get(dis)*(map.get(dis)-1);
    }
    return res;
}

//numberOfBoomerange1 的空间复杂度是O(n^2^)
//稍作修改可以把空间复杂度降低成O（n）
//也就是把map放在循环体内
public  int  numberOfBoomerange2(int[][] points){
    int res = 0;
   
    for( int i = 0 ;i <points.length;i++){
		 HashMap<Integer ,Integer> map = new HashMap();
        for(int  j  = 0; j<points.length;j++){
            int dis = pointDis(points[i],points[j]);
            if(dis == 0) break;
            if(map.containsKey(dis)){
                map.put(dis,1+map.get(dis));
            }else{
                map.put(dis,1);
            }
        }
         //A ~n~ ^2^
   		 for(int dis: map){
			res +=  map.get(dis)*(map.get(dis)-1);
    	}
    }
    
   
    return res;
}

public int pointDis(int indexA ,int indexB){
    return  result;
}
```



#### 存在重复元素II

题号：219

![image-20200725160416050](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142146.png)

思路： 

- 看到这个i和 j 的差的绝对值之多为k ,就应该想到用窗口
  把窗口下的全部的值都放入查找表中



```java
public  boolean  containsNearbyDuplicate(int[]  nums ,int k){
    if(nums == null || nums.length) reture  false;
    if(k<=0)  return false;
    TreeSet<Integer> set = new TreeSet();
    for(int i = 0; i< nums.length;i++){
        if(set.contain(nums[i])){
            return true;
        }else{
            set.add(nums[i]);
            //这里就是维系窗口大小的地方
            if(set.size()>=k+1){
                set.remove(nums[i-k]);
            }
        }
        
    }
    return false;
}
```

#### 存在重复元素III

题号：220

![image-20200725170132748](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142152.png)

思路：

-  和 219提类似，但是本题是两个元素的差的绝对值要小于t
  所以需要借助treeset的ceiling() 方法 ceiling方法是用来获取最接近这个上限的元素。
- 注意是绝对值



```java
public boolean containNearByAlmostDuplicate(int[] nums,int k , t){
    if(nums == null ||nums.length<=0) return false;
    if(k<=0) return false;
    
    TreeSet<Integer> set = new  TreeSet();
    // ceiling 就是获取大于 nums[i]-t的最大元素
    for(int i  = 0 ; i <nums.length ; i++){
        if(set.ceiling(nums[i]-t)!=null&&
          set.ceiling(nums[i]-t)<=(nums[i]+t)){
            return true;
        }
        set.add(nums[i]);
        // 维护窗口
        if(set.size()==k+1){
            set.remove(nums[i-k]);
        }
    }
    return false;
    
    
    
}
```



#### 反转链表

题号：206

![image-20200727154744755](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142200.png)



思路：

一般是不会允许使用栈的， 要操作指针

```java
public class ListNode{
    int val ;
    ListNode next;
    ListNode(int x){
        val = x;
    }
    
    public ListNode reverserList(ListNode node){
        ListNode cur = node;
        ListNode pre = null;

        while(cur !=null){
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur ;
            cur = next;
        }
        
        return  pre;
    }
}
```





#### 移除链表元素

题号：203

![image-20200729153700725](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142206.png)

```java
public ListNode  removeElements(ListNode node ,int value){
    ListNode dummyHead  = new ListNode(-1 ,node);
	ListNode curNode = dummyHead ;
    while(curNode.next!=null){
        if(curNode.next.getValue() == value){
            ListNode tmp = curNode.next;
            curNode.next = tmp.curNode.next.next;
            // curNode 本身被改变 所以 不需要移动curNode本身
        }else{
        	curNode = curNode.next;
        }
    }
    return dummyHead.next;
}
```

#### 两两交换链表中的节点

题号：24

![image-20200730142650462](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142214.png)

思路：
一定要画图理清楚，
要增加个虚拟头结点（哨兵），这样可以忽略头节点的变化



```java
 public ListNode swapPairs(ListNode head) {
        if(head == null ||head.next ==null)
             return head;
    ListNode dummyHead = new ListNode(0);
    dummyHead.next = head;
    
    ListNode cur = dummyHead;
    ListNode next = cur.next;
    ListNode doubleNext =  next.next;
    // dummyHead.next = next.next;
    while(cur!=null&&next!=null&&doubleNext!=null){  
        cur.next = doubleNext;
        next.next =doubleNext.next;
        doubleNext.next = next;    
        cur = next;
        if(cur!=null){
            next = cur.next;
        }
        if(next!=null){
            doubleNext = next.next;
        }

    }
    return  dummyHead.next;
    }
```

#### 删除链表中的节点



![image-20200731115835162](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142220.png)



思路：

注意题意是只知道要删除的节点，并不知道要删除节点的跟节点，以及上一个节点；

因此采用的删除就是把下一个节点的值赋给自己，然自己成为“下一个节点”。

```java
public void deleteNote(ListNode node){
    if(node==null||node.next ==null)
        throw IllegalArgumentException("");
    node.value = node.next.value;
    node.next = node.next.next;
}
```



#### 删除表的倒数第N个节点 并且返回链表的头结点

![image-20200731150755066](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142240.png)

思路：
一般要知道这个链表多长，才可以知道倒数第N个节点的序号
所以得先遍历一遍这个链表，知道长度了之后，再做处理。

但是我们也可以借助 窗口（窗口的长度是N）的概念，这样一次遍历就可以得到倒数第N个节点

```java
public ListNode removeNthFromEnd(ListNode head){
    if(head  == null) return null;
    
    //添加N个头节点
    ListNode dummyHead = head;
    for(int i = 0 ; i<=N ;i++){
		ListNode tmp = new ListNode(0);
        tmp.next=dummyHead;
        dummyHead = tmp;
    }
    
    ListNode left  = dummyHead;
    LisstNode right = head;
    
	while(right !=null){
        left = left.next;
        right = right.next;
    }
   
    right.next = right.next.next;
    
    return head;
}
```

#### 有效的括号

![image-20200803141108286](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142248.png)



思路： 
很明显用个栈，遇到左括号时入栈， 遇到右括号时出栈并检查是否匹配
char和Character之间的转化是自动拆装包的。

```java
public boolean isValid(String strs){
    Stack<Character> stack = new Stack();
    
    for(int i = 0 ; i<strs.length(); i++){
        
        if(strs.charAt(i) == '{'||
           strs.charAt(i) == '['||
           strs.charAt(i) == '('
          ){
            stack.push(strs.charAt(i));
        }else{
			if(stack.size() == null){
				return false;
            }
            Character c = stack.pop();
            Character target  ='';
            
            if(strs.charAt(i) == '}'){
            	target = '{'    ;
            }else if(strs.charAt(i) == ']'){
                target = '[';
            }else if(strs.charAt(i) == ')'){
                target = '(';
            }
            

		    if(c != target ){
                return false;   
            }
            
            
        }
       return (stack.size()==0);
    }
   
}


```





#### 二叉树中序遍历

题号：94

递归：

```java
public List<Integer> inorederTraversal(TreeNode root){
    if(root == null ) return null;
    return tranversal(root);
}

public List<Integer> tranversal(TreeeNode node){
	//  addAll 不允许传 null 所以new 个arrayList
    if（node == null ） return  new ArrayList();
    List<Integer> result = new ArrayList();
	
    result.addAll(traversal(node.left));
    result.add(node);
    result.addAll(traversal.right);
    
    return result;
    
}

```

非递归

1. 还是得借助栈来完成
2. 栈里保存两中操作， scan 到该数据和需要查询该数据
3. 每当scan到该数据的操作出栈时，就要转化为scan right ，scan left ， 以及查询该数据 三个操作入栈

```java
  public List<Integer> inorderTraversal(TreeNode root) {

      if(root == null ) return  new ArrayList();
    
        List<Integer> result = new ArrayList();
        Stack<Command> stack = new Stack();
        stack.push(new Command("scan",root));
        while(!stack.empty()){
            Command cmd  = stack.pop();
            
            if(cmd.cmd.equals("get")){
                result.add(cmd.node.val);
            }else{
                if(cmd.node.right!=null){
                stack.push(new Command("scan",cmd.node.right));
                }
                stack.push(new Command("get",cmd.node));
                if(cmd.node.left!=null){
                    stack.push(new Command("scan",cmd.node.left));
                }
            }

        }
        return result;
    }
    


    class Command{
        String  cmd ;//scan , get
        TreeNode node;
        public Command(String cmd ,TreeNode node){
            this.cmd = cmd;
            this.node = node;
        }
    }
```



#### 二叉树的前序遍历



题号：144

```java
public List<Integer> inorederTraversal(TreeNode root){
    if(root == null ) return null;
    return tranversal(root);
}

public List<Integer> tranversal(TreeeNode node){
	//  addAll 不允许传 null 所以new 个arrayList
    if（node == null ） return  new ArrayList();
    List<Integer> result = new ArrayList();
	
    result.add(node);
    result.addAll(traversal(node.left));
    result.addAll(traversal.right);
    
    return result;
    
}

```



非递归：

参考中序

```JAVA
  public List<Integer> preorderTraversal(TreeNode root) {

      if(root == null ) return  new ArrayList();
    
        List<Integer> result = new ArrayList();
        Stack<Command> stack = new Stack();
        stack.push(new Command("scan",root));
        while(!stack.empty()){
            Command cmd  = stack.pop();
            
            if(cmd.cmd.equals("get")){
                result.add(cmd.node.val);
            }else{
                
                if(cmd.node.right!=null){
                stack.push(new Command("scan",cmd.node.right));
                }
              
                if(cmd.node.left!=null){
                    stack.push(new Command("scan",cmd.node.left));
                }
                
                 stack.push(new  Command("get",cmd.node));
            }

        }
        return result;
    }
    


    class Command{
        String  cmd ;//scan , get
        TreeNode node;
        public Command(String cmd ,TreeNode node){
            this.cmd = cmd;
            this.node = node;
        }
    }
```



#### 二叉树的后序遍历

题号：145

```java
public List<Integer> inorederTraversal(TreeNode root){
    if(root == null ) return null;
    return tranversal(root);
}

public List<Integer> tranversal(TreeeNode node){
	//  addAll 不允许传 null 所以new 个arrayList
    if（node == null ） return  new ArrayList();
    List<Integer> result = new ArrayList();
	
    
    result.addAll(traversal(node.left));
    result.addAll(traversal.right);
    result.add(node);
    return result;
    
}

```

非递归：

参考中序遍历

```java
  public List<Integer> postorderTraversal(TreeNode root) {

      if(root == null ) return  new ArrayList();
    
        List<Integer> result = new ArrayList();
        Stack<Command> stack = new Stack();
        stack.push(new Command("scan",root));
        while(!stack.empty()){
            Command cmd  = stack.pop();
            
            if(cmd.cmd.equals("get")){
                result.add(cmd.node.val);
            }else{
                
                 stack.push(new  Command("get",cmd.node));
                
                if(cmd.node.right!=null){
                stack.push(new Command("scan",cmd.node.right));
                }
              
                if(cmd.node.left!=null){
                    stack.push(new Command("scan",cmd.node.left));
                }
                
                
            }

        }
        return result;
    }
    


    class Command{
        String  cmd ;//scan , get
        TreeNode node;
        public Command(String cmd ,TreeNode node){
            this.cmd = cmd;
            this.node = node;
        }
    }
```



#### 二叉树层序遍历

题号：102

![image-20200806143448550](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142302.png)

思路
注意，有层级的打印
所以需要用Pair来 存储层级信息



借助队列 ，参考中序遍历的非递归实现

```java
 public List<List<Integer>> levelOrder(TreeNode root) {

    
	if( root == null) return new ArrayList();
    
    LinkedList<Pair> queue = new  LinkedList();
    
    queue.push(new Pair(root,0));
    List<List<Integer>> result = new ArrayList();
    while(!queue.isEmpty()){
        Pair  pair = queue.removeFirst();
       
        int level = pair.getLevel();
        if(level==result.size()){
            result.add(new ArrayList());
        }
        result.get(level).add(pair.getNode().val);
        if(pair.getNode().left!=null){
        	queue.add(new Pair(pair.getNode().left,level+1));
           
        }
        if(pair.getNode().right!=null){
            queue.add(new Pair(pair.getNode().right,level+1));
        }
    }
    return result;
 
}

class Pair{
    TreeNode node ;
    int level;
    public Pair(TreeNode node ,int level){
        this.node  = node;
        this.level = level;
    }

    public TreeNode getNode(){
        return node;
    }
    public int getLevel(){
        return level;
    }
}
```

#### 完全平方数



![image-20200806163509806](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142313.png)

思路：
图的最小路径的思路

另外 ，这里面要返回的时候个数最少是多少个 ，而不是返回具体的路径。

```java
public int numSquares(int n){
    LinkedList<Pair<Integer,Integer>>  queue = new LinkedList();
    queue.addLast(new Pair(n,0));
    
    // 避免重复添加
    boolean[] visited = new boolean[n+1];
    visited[n] = true;
    while(!queue.isEmpty()){
        Pair tmpP = queue.removeFirst();
        int value = tmpP.getKey();
        int step = tmpP.getVaule();
        
        if(value == 0 ) return step;
        
        
		for(int i = 1 ;value - i*i>=0 ;i++){
			int a = value - i*i；
            if(a == 0 ) return step +1;
            
            if(!visited[a]){
            queue.addLast(new Pair(a,step+1 ));
                visited[tmp] =true;
            }
        }
        
    }
    
    return  -1;
}
```



#### 前K个高频元素

![image-20200811091939627](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142319.png)

思路：

优先队列

```java
  public int[] topKFrequent(int[] nums, int k) {
         if(nums == null) return null;
    
    //首先统计一下 各个元素出现频率
    // 怎么统计呢？ 用一个hashMap
	HashMap<Integer,Integer> fre = new HashMap();
    for(int i = 0 ; i<nums.length ;i++){
		if(fre.containsKey(nums[i])){
            fre.put(nums[i],fre.get(nums[i])+1);   
        }else{
            fre.put(nums[i],1);
        }
    }
    
    //自定义优先队列
    PriorityQueue<Pair<Integer, Integer>> queue = new PriorityQueue<Pair<Integer, Integer>>(new PairComparator());
    for(int key:fre.keySet()){
        if(queue.size() >= k){
            if(queue.peek().getKey()<fre.get(key)){
                queue.poll();
                queue.add(new Pair(fre.get(key),key));
            }
        }else{
            queue.add(new Pair(fre.get(key),key));
        }
    }
    int[] result  = new int[k];
    for(int i = 0 ; i<k ;i++){
        result[i] = queue.poll().getValue();
    }
    return result;

    }

    private class PairComparator implements Comparator<Pair<Integer, Integer>>{

        @Override
        public int compare(Pair<Integer, Integer> p1, Pair<Integer, Integer> p2){
            if(p1.getKey() != p2.getKey())
                return p1.getKey() - p2.getKey();
            return p1.getValue() - p2.getValue();
        }
    }
```

#### 二叉树的最大深度



![image-20200811114757639](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142333.png)

思路
递归  最大成熟就是 左右子树中最大层数+1；

```java
public int maxDepth(TreeNode root) {
	if(root == null ) return 0;
    
    return 1+	Math.max(maxDepth(root.right),maxDepth(root.left));
       
}
```



#### 翻转二叉树

![image-20200811150344505](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142339.png)

google 淘汰一个大神的题

```java
public  TreeNode  solution(TreeNode root ){
	if( root == null ) return null;

    return reverse(root);
}

public TreeNode reverse(TreeNode root){
	if(root  == null ) return  null;
    TreeNode tmp = root.right;
    root.right = reverse(root.left);
    root.left = reverse(tmp);
    return root;
}
```



#### 路径总和

![image-20200811154728193](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142345.png)

思路：
遍历 到子节点的时候就计算一下 和 是否为目标值
并且查看下当前节点是不是叶子节点。

```java
public boolean solution(TreeNode root ,int targetValue){
    if(root == null) return false;
    
    if(targetValur == root.val&&root.left == null &&root.right == null )  return true;
    
	return solution(root.left,targetValue - root.val)||solution(root.right,targetValue - root.val);    
}


```



#### 二叉树的所有路径

![image-20200811171553129](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142352.png)

思路：

1. 把左右子节点的路径加起来就是总的路径
2. 遍历节点的时候 ，返回的是路径



```java
public List<String> solution(TreeNode root){
	if(root == null) return new ArrayList();
    
    if( root.left == null && root.right == null){
		List<String> result = new ArrayList();
        return result.add(""+root.val);
    }
    
    List<String> result = new ArrayList();
    
    List<String> leftResult = solution(root.left);     List<String> rightResult = solution(root.right);
    for(String str : leftResult){
        String tmp = root.val+"->"+str;
	    result.add(tmp);        
    }
        for(String str : rightResult){
        String tmp = root.val+"->"+str;
	    result.add(tmp);        
    }
  
    return result;
}


```

#### 路径总和2

![image-20200812162750456](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142401.png)

思路：
分一下情况：
先找到本节点的全部数量
再找左右子节点的全部数量
递归 相加
最终结果就是 解

问题是怎么求本节点的全部数量


1. 首先先看自己是不是，
   是的话就 +1， 然后res+1;
   然后看子节点是 中和是 0 的路径：
2. 如果不是，则看子节点中 num - node.val的全部路径





```java
public int solution(TreeNode root,int sum ){
	if(root == null ) return  0;
    return 0;
}
```





#### 电话号码的字母组合







![image-20200818162203037](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200818162203.png)



思路：
获取输入串的每一个字符的每一个可能，然后承接上一个字符输出的字符串

```java
   private String letterMap[] = {
            " ",    //0
            "",     //1
            "abc",  //2
            "def",  //3
            "ghi",  //4
            "jkl",  //5
            "mno",  //6
            "pqrs", //7
            "tuv",  //8
            "wxyz"  //9
    };

    private ArrayList<String> res;

    public List<String> letterCombinations(String digits) {
        res = new ArrayList();
        if(digits ==null) return res;

        findCombination(digits,0,"");
        return res;
    }


    private void findCombination(String digits, int index, String s){
//        if(!s.isEmpty()){
//            res.add(s);
//        }
        if(index == digits.length()) {
            res.add(s);
            return ;
        }

        char c = digits.charAt(index);
        String  tmpS = letterMap[c-'0'];
        for(int i = 0 ; i< tmpS.length();i++){
            findCombination(digits,index+1,s+tmpS.charAt(i));
        }
        return ;
    }
```



#### 全排列

![image-20200819183202265](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200819183202.png)





#### 组合

![image-20200819161516940](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200819161517.png)

思路：
题意其实就是要求输出C~n~^k^  的所有可能

要注意 对于组合顺序是没有意义的。

```java
public class LeetCode_77 {

    public static void main(String[] args) {
        System.out.println("Hello World!");
        LeetCode_77  code = new LeetCode_77();
        System.out.println(code.combine(4,2));

    }


    List<List<Integer>> res = new ArrayList();
    public List<List<Integer>> combine(int n ,int k ){
        res = new ArrayList<List<Integer>>();
        LinkedList<Integer> c  = new LinkedList();
        solution(n,k,1,c);
        return res;

    }

    private void solution(int n ,int k ,int start ,LinkedList<Integer> c){
        if(c.size() == k) {
            //java是值传递， 但是对于对象来说是地址传递，所以内外层的c是会相互影响的 
            //外层中，c是会回溯的。所以这里用clone来赋值
            res.add((List<Integer>) c.clone());
            return ;
        }

        // 核心在这
        // 都从自己的后面取 能保证没有重复的。
        // 但其实还是不太理解怎么就保证没重复的了
        // 因为是组合 组合是不在乎 顺序的
        // 比如 1-3-4 和 1-4-3  是一样的。
		// 在 1-3 的时候 是肯定遍历到1-3-4了的

        for(int i = start ;i<=n;i++){
            c.addLast(i);
            solution(n,k,i+1,c);
//            这里实现回溯
            c.removeLast();
        }
    }
}
```

优化  减枝

如果说 还需要  5 个数字 ，但是剩下只有4个数字了。那就说明 剩下的的数字无论怎么组合都组合不出来了。

        for(int i = start ;i<=n;i++){
            c.addLast(i);
            solution(n,k,i+1,c);
            //            这里实现回溯
                c.removeLast();
            }
            
         所以I的有效取值范围是：
         start -> n - (k -c.size()-1)
            
         --这里可以优化成 
         for(int i = start ; i<= n -(k-c.size())+1;i++)
#### 全排列

![image-20200821100701482](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200821100701.png)

思路：

A~n~^n^的全部可能

```java
  List<List<Integer>>  result = new ArrayList();
    public List<List<Integer>>  solution(int[] arr){
        List list = new ArrayList();

        for(int i = 1 ; i<=arr.length ;i++){
            list.add(i);
        }

        solution(new ArrayList(),list);
        return result;
    }

    public void solution(/*当前已经排了序的列表*/List<Integer> tmp,/*还未排序的元素的列表*/List<Integer> list){

        if(list.size()==0){
            result.add(new ArrayList<>(tmp));
            return ;
        }
        // 不能再原本list 的遍历中 改list内部
        List<Integer> tmpList = new ArrayList<>(list);

        for(Integer i :tmpList){
            list.remove((Object)i);
            tmp.add(i);
            solution(tmp,list);
            // 回溯
            tmp.remove((Object)i);
            list.add(i);
        }
        return ;
    }
```



#### 单词搜索

![image-20200821111332486](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200821111332.png)

思路：
回溯

```java
import com.sun.xml.internal.ws.util.StringUtils;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class LeetCode_79 {

    public static void main(String[] args) {
        char[][] b1 = {
                {'A','B','C','E'},
                {'S','F','C','S'},
                {'A','D','E','E'}};
        System.out.println("Hello World!");
        LeetCode_79 code = new LeetCode_79();
//        String words[] = {"ABCCED", "SEE", "ABCB" };
//        String words[] = {"ABCCED" };
//        String words[] = {"SEE" };
//        String words[] = {"SE" };
        String words[] = {"SE"};
        for(int i = 0 ; i < words.length ; i ++)
            if((new LeetCode_79()).solution(b1, words[i]))
                System.out.println("found " + words[i]);
            else
                System.out.println("can not found " + words[i]);
//        System.out.println(code.solution(b1,2));

    }


    int m ; // 行数
    int n ; // 列数
    boolean[][]  isEnableRead ;

    public boolean solution(char[][] arr,String word){

        if(arr == null || arr.length<=0) return false;

        if(word==null||word.length()<=0)  throw new  IllegalArgumentException(" illegal  params");

        m  = arr.length; //行数
        n  = arr[0].length;// 列数

        isEnableRead = new boolean[m][n];

        for(boolean[] t : isEnableRead){
            for(boolean t2 :t){
                t2 = true;
            }

        }

        for(int i = 0; i<m ; i++)
            for(int j = 0 ;j <n;j++) {
                if(searchWord(arr, i, j, 0, word))
                    return true;
            }

        return false;
    }

    public boolean  searchWord(char[][] arr,final int i,final int j , int index,String word){


        if(!validData(i,j))  return false;

        if(index == word.length()-1)  return true;

        System.out.println(" " + arr[i][j]);

        
        if(arr[i][j] == word.charAt(index)){

            System.out.println(" == " + arr[i][j]);
            // 上
            if(validData(i-1,j)){
                isEnableRead[i-1][j] = false;
                boolean result = searchWord(arr,i-1,j,index+1,word);
                System.out.println("top == " + arr[i-1][j]);
                isEnableRead[i-1][j] = true;
                if(result)  return  true;
            }



            // 右
            if(validData(i,j+1)){
                isEnableRead[i][j+1] = false;
                boolean result = searchWord(arr,i,j+1,index+1,word);
                isEnableRead[i][j+1] = true;
                System.out.println("right  == " + arr[i][j+1]);

                if(result)  return  true;
            }



            // 下
            if(validData(i+1,j)){
                isEnableRead[i+1][j] = false;
                boolean result = searchWord(arr,i+1,j,index+1,word);
                isEnableRead[i+1][j] = true;
                System.out.println("bottom == " + arr[i+1][j]);

                if(result)  return  true;
            }

            // 左
            if(validData(i,j-1)){
                isEnableRead[i][j-1] = false;
                boolean result = searchWord(arr,i,j-1,index+1,word);
                isEnableRead[i][j-1] = true;
                System.out.println("left == " + arr[i][j-1]);

                if(result)  return  true;
            }


        }

        return false;
    }

    private boolean validData(int i, int j ) {

        return i>=0 &&j>=0 && i <m && j<n;
    }

}

```



优化： 用数组来模拟上下左右的移动的切换

```java
    int[][] d = {{-1,0},{0,+1},{+1,0},{0,-1}};// 上右下左的坐标切换


    public boolean  searchWord2(char[][] arr,final int i,final int j , int index,String word){
        if(index == word.length())  return true;

        if(!validData(i,j))  return false;

        System.out.println(" " + arr[i][j]);

        if(arr[i][j] == word.charAt(index)){
            isEnableRead[i][j] = false;
            System.out.println(" == " + arr[i][j]);

            for(int[] tmp : d){
                int newi = i+tmp[0];
                int newj = j+tmp[1];
                if(validData(newi,newj)){
                    boolean result = searchWord(arr,newi,newj,index+1,word);
                    System.out.println("top == " + arr[newi][newj]);
                    if(result)  return  true;
                }
            }
            isEnableRead[i][j] = true;


        }

        return false;
    }

```

#### 岛屿的数量

![image-20200824142445405](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824142445.png)

思路：

和单词搜索类似  但是不需要回溯。

```java
public class LeetCode_200 {

    public static void main(String[] args) {
        System.out.println("Hello World!");
        LeetCode_200 code = new LeetCode_200();
        char[][] ints  = {

                {'1','1','1','1','0'},
                {'1','1','0','1','0'},
                {'1','1','0','0','0'},
                {'0','0','0','0','0'}
        };



        System.out.println(code.numIslands(ints));

    }


    boolean[][]  isVisited ;
    int m ; //行数
    int n ;// 列数
    public int numIslands(char[][] grid) {

        if(grid == null || grid.length == 0 ||grid[0].length == 0) return 0;

        m  = grid.length; //行数
        n  = grid[0].length;// 列数


        isVisited = new boolean[m][n];

        int result = 0;
        for(int i = 0 ;i<m ;i++)
            for(int j = 0 ; j<n;j++)
                if(isArea(i,j)&&grid[i][j]=='1'&&!isVisited[i][j]) {
                    System.out.println("-- i "+i +"  -- j "+ j );

                    handle(grid,i,j);
                    result++;
                }

        return  result;
    }


    int[][]  d = {{-1,0},{0,+1},{1,0},{0,-1}};


    private void handle(char[][] grid,int i, int j) {

        if(!isArea(i,j)) return;

        if(isVisited[i][j]) return;

        if(grid[i][j] == '0') return;

        isVisited[i][j] = true;
		//向四周蔓延
        for(int p = 0 ; p<4;p++) {
            int newi = i + d[p][0];
            int newj = j + d[p][1];
            handle(grid,newi,newj);
        }
    }

    private boolean isArea(int i ,int j){
        return  i>=0 &&j>=0 &&i<m&&j<n;
    }


}

```

#### N皇后问题

![image-20200824183014587](https://raw.githubusercontent.com/YYQuan/MyTypora/master/img/20200824183015.png)



#### 爬楼梯

![image-20200831180900338](https://i.loli.net/2020/08/31/u4N7YJQyFaUlBIq.png)



思路： 这是一个动态规划的问题。
类似菲波那切数列
f(n) = f(n-1)  +f(n-2);



```java
public int  solution( int n ){
    int[] tmp =  new int[n+1];
    tmp[0]=1;
    tmp[1]=1;
    tmp[2]=2;
    
    if(n==1) return tmp[1];
    
    for(int i =  2 ;i<=n ; i++){
        tmp[i]  = tmp[i-1] +tmp[i-2]
    }
    return tmp[n];
}
```



#### 整数拆分

![image-20200901151912981](https://i.loli.net/2020/09/01/oWPFza9vs73mn6C.png)



思路：
有多解法
一般来说 回溯是 自顶向下，  记忆化搜索也是自顶向下但是把重复过程的答案保存下来了，而动态规划一般是自底向上的。

##### 1.暴力解法

```java
public  int solution(int n ){
    int res = -1;
    if(n == 1 ) return 1;
    if(n == 2 ) return 1;
    for(int i = 2; i<=n ; i++){
        res = Math.max(res,Math.max(i*(n-i),i*solution(n-i)));
	}
    return res;
}
```



##### 记忆化搜索

```java

int[]  tmpResult;
public int solution(int n ){
    tmpResult = new int[n+1];
    Arrays.fill(tmpResult,-1);
    return  0;
}

public int solutionActual(int n ){
    if( n == 1){
      return 1;   
    }
    if( n == 2) return 1;
    
    if( tmpResult[n]!= -1) return tmpResult[n];
    int  res = -1;
    for(int i = 2 ; i <n ;i++)
	     res = Math.max(res, Math.max(i*(n-i),i*solutionActual(n-i)));
    tmpRsule[n] = res;
    return res;
}
```

##### 动态规划

```java
  public  int  solution3(int n ){
        int[]     tmp = new int[n+1];
        Arrays.fill(tmp,-1);
//        tmp[0] =1;
        tmp[1] =1;
        tmp[2] =1;
        for(int i = 2 ; i<= n ;i++){
            for(int j = 1 ; j<i;j++){
                tmp[i] =Math.max(tmp[i], Math.max(j*(i-j),j*tmp[i-j]));
            }
        }
        return tmp[n];

    }
```





#### 打家劫舍

![image-20200902110705353](https://i.loli.net/2020/09/02/LNjFhrsMYDapOAw.png)

思路： 动态规划题

用多种方式来解决



##### 记忆化搜索

我的解法

```java
    int[]  tmpResult ; // 存放解的数组
    public int solution(int[] arr){

        tmpResult = new int[arr.length];
        Arrays.fill(tmpResult,-1);
        return solutionActual(arr,0);
    }

    public int  solutionActual(int[] arr,int start){
        int res  = -1;
//        if(arr== null||arr.length<1||start>arr.length-1) return sum ;
        if(arr== null||arr.length<1||start>arr.length-1) return 0 ;

        if(tmpResult[start] != -1)  return tmpResult[start];

        for(int i = start ; i< arr.length ;i++) {
//            res = Math.max(res,Math.max(arr[start]+solutionActual(arr,start+2,0),arr[start]+solutionActual(arr,start+3,0)));
//            res = Math.max(arr[start]+solutionActual(arr,start+2,0),arr[start]+solutionActual(arr,start+3,0));
            // 对于流程来说，  在一间房子里， 有两种选择， 1 跳一间房  start +2  2 跳 两间房 start +3  ( 跳四中间就可用多偷一间了)
            //res = Math.max(res,Math.max(arr[start]+solutionActual(arr,start+2)),arr[start]+solutionactual(arr.start+3));
            // 完整应该是这个样子的  但是 实际上  solutionActual(arr,start+2) 中是包括了 solutionActual(arr,start+3)的情况的
            // 所以solutionActual(arr,start+2) >= solutionActual(arr,start+3)
            // 因此可以简化成下面这个
            res = Math.max(res,arr[start]+solutionActual(arr,start+2));
        }

        tmpResult[start] = res;
        return tmpResult[start];
    }
```

大佬的解法

```java
 // memo[i] 表示考虑抢劫 nums[i...n) 所能获得的最大收益
    private int[] memo;

    public int rob(int[] nums) {
        memo = new int[nums.length];
        Arrays.fill(memo, -1);
        return tryRob(nums, 0);
    }

    // 考虑抢劫nums[index...nums.size())这个范围的所有房子
    private int tryRob(int[] nums, int index){

        if(index >= nums.length)
            return 0;

        if(memo[index] != -1)
            return memo[index];

        int res = 0;
        for(int i = index ; i < nums.length ; i ++)
            res = Math.max(res, nums[i] + tryRob(nums, i + 2));
        memo[index] = res;
        return res;
    }

```



动态规划

我的动态规划的数据是从左往右 加，   大佬的是 从右 往左加。
感觉我的好理解些

```java
   // 动态规划

    // 要自底向上 来求解
    public  int solution2(int[] arr){
        int[]  tmpResult = new int[arr.length+1];
        Arrays.fill(tmpResult,-1);

        if(arr.length ==1 ) return arr[0];
        if(arr.length ==2 ) return Math.max(arr[0],arr[1]);

        tmpResult[0]  = arr[0];
        tmpResult[1]  = Math.max(arr[0],arr[1]);

        for(int i = 2 ; i<arr.length ; i++){
            tmpResult[i] = Math.max(tmpResult[i-1] ,arr[i] + tmpResult[i-2]);
        }


        return tmpResult[arr.length-1];
    }

```

大佬的动态规划

```java
    public int rob(int[] nums) {

        int n = nums.length;
        if(n == 0)
            return 0;

        // memo[i] 表示考虑抢劫 nums[i...n) 所能获得的最大收益
        int[] memo = new int[nums.length];
        memo[n - 1] = nums[n - 1];
        for(int i = n - 2 ; i >= 0 ; i --)
            for (int j = i; j < n; j++)
                memo[i] = Math.max( memo[i],
                                    nums[j] + (j + 2 < n ? memo[j + 2] : 0));

        return memo[0];
    }
```





#### 01背包问题

非leetcode 原题



![image-20200902172050613](https://i.loli.net/2020/09/02/QH1b7lZfK5xnJYp.png)


求装最大价值物品方案

思路：

对于一个物品而言 只有 两种状态，
装 与不装
F（i,c ） = Math( F(i-1,c),v[i]+ F（i-1,c-w(i))





##### 记忆化搜索

```java
int[][] tmpRes;

public  int solution(int[] w,int[] v,int c){
    if(w == null || v == null ||  c<= 0) return 0;
    tmpRes = new  int[w.length][c+1];
    for(int i = 0 ; i<w.length;i++)
        Arrays.fill(tmpRes[i],-1);
    return maxValue(w,v,v.length-1,c);
}

public int maxValue(int[] w, int[] v,int index,int c ){
    if(w == null || v == null ||  c<= 0) return 0;
    if(index<0)  return 0 ;
    
    if(tmpRes[index][c]> 0 ) return tmpRes[index][c];
    
    int res = maxValue(w,v,index-1,c);
    if(c>= w[index]){
        res = Math.max(res,v[index]+maxValue(w,v,index-1,c-w[index]))
    }
    tmpRes[index][c] = res;
    return res;
}
```

##### 动态规划

要使用动态规划需要注意如下特性

1.当物品不变时，背包容量C 越大，那么最大价值方案 >= C 小的时候
2.当背包容量C不变是， 物品N越多，那么最大价值方案 >= C 小的时候
3.由1 ，2 可以得出  最大价值方案 一定是 C和N 最大的时候的方案

我们可以从 一个物品 开始    从0开始增加容量 得到 最大价值，
然后增加一个物品A，再从0开始增加容量
对于增加的这个物品A后 的最大价值 有两种情况

1.装的下这个物品A，并且加进背包中 ，然后背包容量减小，
这时候的最大价值就是 物品A的价值和 减小后的背包容量最大价值之和，由于我们之前已经保存了容量减小后的没有物品A的最大价值，因此就可以直接读取了
2.不装这个物品A，那么就是直接读取 没有物品A的情况下，改容量的最大价值，这个也是之前已经保存下来了的。

根据这两点 就能够完成动态规划的实现了



```java
public int solution(int[] w int[] v,int c){
    
    if(w == null || v == null || w.length <=0 || v.length<=0 || c<0) {
		throw new IllegalArgumentException()
    }
    
    int[][] tmpRes = new int[w.length][c+1];
    
    for(int i = 0 ; i< w.length ;i++){
        Arrays.fill(tmpRes[i],-1);
    }
  
    for(int i = 0 ; i<= c ; i++){
        tmpRes[0][i] = c>w[i]? v[i]:0;
    }
    
    for(int i = 1 ; i<w.length ; i++)
    	for(int j  = 0 ; j<=c ;j++){
            tmpRes[i][j] = tmpRes[i-1][j];
            if(j>w[i]){
                tmpRes[i][j] = Math.max(tmpRes[i][j],v[i]+tmpRes[i-1][j-w[i]]);
            }
        }
    
    return tmpRes[w.length-1][c];
    
}
```



#### 分割等和子集 - 01背包问题的变型



![image-20200904104242792](https://i.loli.net/2020/09/04/BdDUgTlMECJYtxH.png)

思路： 如题所说这是一个类似01背包的问题
怎么类比概念呢。

每一个元素就是一个物品
数组长度就是总容量， 
题目要求的就是 把总容量分成两个容量， 使得他们的价值相等

分成两个子集 ，并且两个子集的元素和相等。
那么就是在数组中抽取 部分元素 ，其和等于全部元素的一半。

那就是容量是确定的，然后放入物品使得容量装满

实际上逻辑就是 给定一个数组和一个容量值C， 在数组中抽取元素， 使得抽取出来的元素之和等于C。



|      | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1    | 1    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
| 5    | 1    | 0    | 0    | 0    | 1    | 1    | 0    | 0    | 0    | 0    | 0    |
| 11   | 1    | 0    | 0    | 0    | 1    | 1    | 0    | 0    | 0    | 0    | 1    |
| 5    | 1    | 0    | 0    | 0    | 1    | 1    | 0    | 0    | 0    | 1    | 1    |

整体的思路就是  加一个物品 就把  上一个物品 是1的地方， 序号+本物品的重量的值 也置为 1，  再把序号是 本物品 处也置为1 。



记忆化搜索

```java
int[][] tmpResult;
public boolean solution(int[] arr){
    if(arr==null|| arr.length<=0) return false;
    
    int sum = 0;
    for(int i = 0; i<arr.length ;i++){
        sum += arr[i];
    }
    
    if(sum%2 != 0 ) return false;
    
    int target = sum/2;
    
    tmpResult  = new int[arr.length][target+1];
    
    for(int i = 0 ; i<arr.length ;i++){
		Arrays.fill(tmpResult[i],-1);
    }
    
    return  solutionActual(arr,arr.length-1,target);
}

public boolean solutionActual(int[] arr,int index ,int sum){
    if( index <0 ) return false;
    if(sum ==0 ) return true;
    if(tmpResult[index][sum]>=0) 
        return tmpResult[index][sum]>0? true:false;
    
	tmpResult[index] [sum] = solutionActual(arr,index-1,sum)||solutionActual(arr,index-1,sum-arr[index]);
    return tmpResult[index][sum];
}
```



动态规划

```java
public  boolean solution(int[] arr){
    if(arr == null || arr.length <=0)
        return false;
    int sum = 0;
    for(int i = 0;i<arr.length;i++){
        sum+=arr[i];
    }
    
    if(sum%2 != 0 ) return false;
    
    int target = sum/2;
    
    boolean[] tmpResult = new boolean[target+1];
    
    for(int i = 0 ; i<arr.length ;i++){
        if(arr[i] <= tartget){
            tmpResult[arr[i]] = true;
        }

        for(int j = 0 ; j<=tartget;j++){		
            if(tmpResult[j]){
                int index =tmpResult[j]+ arr[i];
                if(index<=target){
                    tmpResult[index] =true;
                }
            }
            
        }
    }
    return tmpResult[target];
}
```



中间的判断逻辑可以改下

```java
for(int i = 0;i<=tmpResult;i++){
    tmpResult[i] = (i==arr[i]);
}
for(int i = 1 ;i <=tmpResult ;i++){
    for(int j = target ;j>0;j--){
        tmpResult[j] = tmpResult[j]||tmpResult[j - arr[i]];
    }
}
```





#### 最长上升子序列



![image-20200907160459889](https://i.loli.net/2020/09/07/zEfcUJ87QnmV4re.png)



思路：
往背包问题上靠，

对于本题目，从右往左来看每一个元素

要这个元素， 然后就不能再要比这个元素的值大的元素了。这个时候要的这个元素是值就有点背包中容量的概念在里面。但是只是只能装小于等于当前容量的物品。

状态转移方程如下：

F(i,c) = max(f(i-1,c),f(i-1,v(i) ) ) 



递归 

纯递归是由个陷阱的， 是不能从后往前递归的。
因为 你在最后位置（index）的时候不知道 index-1时，最大的值是多少（并不是[0...index-1]里数值最大的值， 而是[0... index-1中最长序列的最大值]），  不知道最大的值是多少就没有办法判断 index 这个值是否能加入。

这里不能从后往前 递归的原因就是 后面的递归所需要的参数不能一开始就确定下来。



```java
//递归

    // 从后往前 时 要注意 要加入index   那么 index 的值要 index -1 最长队列中的最大值才行
    // 但是前面的没算出来就不知道index -1 的最长队列中的最大值是多少   所以得改成从前往后


    public  int   solution(int[] arr){
        if( arr == null || arr.length <= 0 ) return 0 ;


        solutionActual(arr,0 , 0 , 0);
        return result;
    }

    int result = 0 ;

    public void solutionActual(int[] arr,int index ,int currentMaxValue ,int currentMaxLength){

        if(index > arr.length-1) {
            result = Math.max(result,currentMaxLength);
            return ;
        }

        int currentValue = arr[index];

        if(currentValue>currentMaxValue){
            //加进去
            solutionActual(arr,index+1,currentValue,currentMaxLength+1);
            solutionActual(arr,index+1,currentMaxValue,currentMaxLength);
        }else{
            solutionActual(arr,index+1,currentMaxValue,currentMaxLength);

        }

    }


```



// 我的记忆化搜索

```java
  public int solution2(int[] arr){
        if(arr == null || arr.length <=0) return 0;

        int[] tmpResult = new int[arr.length];
        Arrays.fill(tmpResult,-1);
        // 先从左开始 逐个添加元素
        // 得先知道了前面的最大长度的值， 才能知道后面的
        for(int i = 0 ; i<arr.length ; i++){
            tmpResult[i] =  solutionActual2(arr,i,tmpResult);
        }
        // 注意  最大值不一定是tmpResult[tmpResult.length -1]
        int res = 1;
        for(int i = 0 ; i<tmpResult.length; i++){
            res = Math.max(res,tmpResult[i]);
        }
        return res ;
    }

    // 返回 arr 中 [ 0...index] 的 最大长度
    public int solutionActual2(int[] arr,int index,int[] indexLength){
        int res = 1;
        for(int i = 0 ; i<index ;i++){
            if(arr[index]>arr[i]){
                res = Math.max(res,1+indexLength[i]);
            }
        }
        return res;
    }

```

动态规划

```java
  // 感觉和 记忆化搜索的代码类似
    public int  solutionDync(int[] arr){
        int res= 0 ;

        if(arr == null || arr.length <=0 )  return 0 ;

        int[] tmpResult = new int[arr.length];

        Arrays.fill(tmpResult,-1);
        tmpResult[0] = 1;

        for(int i = 1 ; i< arr.length ;i++){
            int resIndex = 1;
            for(int j = 0 ; j<i ; j++){
                if(arr[i] > arr[j]){
                    resIndex = Math.max(resIndex, 1+tmpResult[j]);
                }
            }
            tmpResult[i] = resIndex;
            res =Math.max(res, resIndex);
        }

        return  res;
    }
```





#### LCS 问题  - 最长公共子序列

有两个字符串S~1~,S~2~， 求最长公共序列长度 (longest_common_subsequence )

首先要搞清楚，这个公共序列是不需要连续的。

eg. 

S~1~ = "adbc"
S~2~= "ac"
那么结果就是 “ac"

思路：

l~1~、l~2~(S~1~、S~2~的长度) 
从后往前，如果S~1~[l~1~-1]    == S~2~[l~2~-1] ,
那么 res +1 ;

如果S~1~[l~1~-1]    ！= S~2~[l~2~-1] 

那么
res  = Math.max(F(S~1~[0...l~1~-1],S~2~[0...l~2~]),S~1~[0...l~1~],S~2~[0...l~2~-1]))

状态转义方程就是:
F(S~1~[l~1~],S~2~[l~2~]) =  F(S~1~[l~1~-1],S~2~[l~2~]) +1;
                        =  Math.max(F(S~1~[l~1~-1],S~2~[l~2~]),S~1~[l~1~],S~2~[l~2~-1]))



递归

```java
    public String solution(String s1 , String s2){

        if(s1 == null || s2 == null || s1.length()<= 0 || s2.length()<= 2){
            throw new IllegalArgumentException();
        }
        return solutionActual(s1,s2,s1.length()-1,s2.length()-1);
    }

    public  String solutionActual(String s1,String s2 ,int indexS1  , int indexS2){
        StringBuilder builder = new StringBuilder();

        if(indexS1 < 0 ||indexS2 <0 ){
            return "";
        }
        if(s1.charAt(indexS1) == s2.charAt(indexS2)){


            return  solutionActual(s1,s2,indexS1-1,indexS2-1) +s1.charAt(indexS1);
        }else{
            String str1 = solutionActual(s1,s2,indexS1-1,indexS2);
            String str2 = solutionActual(s1,s2,indexS1,indexS2-1);


            return str1.length()>str2.length() ?str1:str2;
        }

    }
```



递归的思路还是挺简单的。
但是来分析下其时间复杂度。
应该是 2^n^
实际测试的时候发现 递归慢的令人发指

所以要优化下。
先看看能不能记忆化一下



记忆化搜索

```java
// 尝试记忆化
    String[][] tmpResult ;
    public String solution2(String s1 , String s2){

        if(s1 == null || s2 == null || s1.length()<= 0 || s2.length()<= 2){
            throw new IllegalArgumentException();
        }
        tmpResult = new String[s1.length()][s2.length()];

        return solutionActual2(s1,s2,s1.length()-1,s2.length()-1);
    }

    public  String solutionActual2(String s1,String s2 ,int indexS1  , int indexS2) {
        if (indexS1 < 0 || indexS2 < 0) {
            return "";
        }
        if (tmpResult[indexS1][indexS2] != null && tmpResult[indexS1][indexS2].length() > 0)
            return tmpResult[indexS1][indexS2];
        if (s1.charAt(indexS1) == s2.charAt(indexS2)) {
            tmpResult[indexS1][indexS2] = solutionActual2(s1, s2, indexS1 - 1, indexS2 - 1) + s1.charAt(indexS1);
        } else {
            if(indexS1>0&&indexS2>0) {
                tmpResult[indexS1 - 1][indexS2] = solutionActual2(s1, s2, indexS1 - 1, indexS2);
                tmpResult[indexS1][indexS2 - 1] = solutionActual2(s1, s2, indexS1, indexS2 - 1);
                tmpResult[indexS1][indexS2] = tmpResult[indexS1 - 1][indexS2].length() > tmpResult[indexS1][indexS2 - 1].length() ?
                        tmpResult[indexS1 - 1][indexS2] : tmpResult[indexS1][indexS2 - 1];

            }else if(indexS1>0){
                tmpResult[indexS1 - 1][indexS2] = solutionActual2(s1, s2, indexS1 - 1, indexS2);
                tmpResult[indexS1][indexS2] = tmpResult[indexS1 - 1][indexS2];
            }else if(indexS1<0){
                tmpResult[indexS1 ][indexS2-1] = solutionActual2(s1, s2, indexS1 - 1, indexS2);
                tmpResult[indexS1][indexS2] = tmpResult[indexS1 ][indexS2-1];
            }else{
                tmpResult[indexS1][indexS2] = "";
            }
        }
        return tmpResult[indexS1][indexS2];
    }
```



动态规划
画下示意图
以S~1~= ”aabcd" ,S~2~ = "aad"为例

x = S~2~.length = 3;

y = S~1~.length = 5

初始状态如下，

| 1    | 1    | 1    |
| ---- | ---- | ---- |
| 1    |      |      |
| 1    |      |      |
| 1    |      |      |
| 1    |      |      |

其实也就是tmp[l~1~] [l~2~] 中 
当中S~1~[x] == S~2~[0] 
那么tmp[k] [0] = 1 k>x
当中S~2~[y] == S~1~[0]
那么tmp[0] [q] =1  q>y

接着 tmp[n] [m] 的其他值 再按照 

1. 如果S~1~[n] == S~2~[m] 
   那么 找  tmp[n-1] [m] 和tmp[n] [m-1] 的大的那个值来 +1 ；
2. 如果S~1~[n] == S~2~[m] 
   那么 找  tmp[n-1] [m] 和tmp[n] [m-1] 的大的那个值

按照这个逻辑就能得到 一个最长公共序列的长度的表。
要得到最终的最长公共序列的具体元素，通过这个表就能得到。

tmp[x~1~] [ y~1~] >Math.max(tmp[x~1~-1] [y~1~],tmp[x~1~] [y~1~-1]); 那么 S~2~[x~1~](也就是S~1~[y~1~ ]) 是最长公共子序列当中的元素

难点：怎么恢复？
看下图理解下。

![image-20200909172624011](https://i.loli.net/2020/09/09/6n3xukeYwCRJ41V.png)

```java

```



从表格来说，其实就是 找到在S~1~S~2~的index增加的过程中，第一个value值，也就是最左上角的那个值，优先左。

恢复的特征：

1. s1.charAt(m) == s2.charAt(n)
2. 只能向  length 减小处移动（即只能向左/上移动）， 优先向左 移动（是必须优先向左）
3. 路径要连续 ，因为 序列中的顺序是由限制的![image-20200909184704192](https://i.loli.net/2020/09/09/zMtAks7x9gFCIwu.png)

```java
// 通过memo反向求解s1和s2的最长公共子序列
        m = s1.length() - 1;
        n = s2.length() - 1;
        StringBuilder res = new StringBuilder("");
        while(m >= 0 && n >= 0)
            if(s1.charAt(m) == s2.charAt(n)){
                res.insert(0, s1.charAt(m));
                m --;
                n --;
            }
            else if(m == 0)
                n --;
            else if(n == 0)
                m --;
            else{
                if(memo[m-1][n] > memo[m][n-1])
                    m --;
                else
                    n --;
            }

        return res.toString();
```

  s1 = "AAACCGTGAGTTATTCGTTCTAGAA";
   s2 = "CACCCCTAAGGTACCTTTGGTTC";

的图解

![image-20200910112757629](https://i.loli.net/2020/09/10/6c9NmInJEb7qHQ8.png)

优先往左

![image-20200910140939468](https://i.loli.net/2020/09/10/CQIduo8rRgb6liW.png)

```

123456789  123456789  1234567
CACCCCTAAG GTACCTTTGG TTC
AAACCGTGAG TTATTCGTTC TAGAA

A

ACCTAGTATTGTTC
```




优先往右

![image-20200910165404928](https://i.loli.net/2020/09/10/eIzZGobWTnvqam2.png)

