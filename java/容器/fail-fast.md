## 增强for循环

for (String str : list)  {}

增强for循环是java的语法糖，实际上是通过iterator迭代器遍历容器。

下面两段代码，Java的字节码指令基本上是一样的，第一个for-each，第二个iterator迭代器

```java
List<String> listA = new ArrayList<String>();
        for(String str : listA) {
            System.err.println(str);
        }

List<String> listA = new ArrayList<String>();
        for (Iterator<String> iter = listA.iterator(); iter.hasNext();) {
            String s = iter.next();
            System.err.println(s);

```



当使用for-each遍历容器进行删除操作时，会出现fail-fast错误即ConcurrentModificationException



## fail-fast

使用容器的iterator遍历并删除数据时，会抛出ConcurrentModificationException，也就是fail-fast异常



java容器都自己实现了迭代器iterator（常用的List、Map），迭代器的next()方法获取下一个元素时首先校验当前容器元素的个数（modCount）是否符合预期个数（expectedModCount），如果容器元素个数与预期个数不相等modCount  !=  expectedModCount，则抛出并发修改异常（ConcurrentModificationException），也就是fail-fast异常



ArrayList源码：

```java

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            // 校验容器数量
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
		// fail-fast 校验数据量是否相同，不同抛出ConcurrentModificationException
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```





## 解决

### fori 需要手动修正下标



删除remove()之后，容器将被删除之后的数据整体向前挪动1个下标，如果不修正下标，会漏掉连续的第二个需要删除的元素如下

```java
List<String> platformList = new ArrayList<>();
platformList.add("博客园");
platformList.add("博客园");
platformList.add("CSDN");
platformList.add("掘金");

for (int i = 0; i < platformList.size(); i++) {
    String item = platformList.get(i);
    if ("博客园".equals(item)) {
        platformList.remove(i);
        //修正下标，
        i = i - 1;
    }
}

System.out.println(platformList);
```



### removeIf



platformList.removeIf(platform -> "博客园".equals(platform));

removeIf()方法 删除元素不是在原来的容器中操作，而是使用BitSet将要删除元素的下标记录下来，然后容器中删除元素后一个元素向前移动。

```java
@Override
    public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }
```

