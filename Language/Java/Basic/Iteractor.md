# Basic
### 迭代器Iterator

    迭代器是一种设计模式，Iterator是一个迭代器对象，可以用来遍历集合对象。
    ```
    // <E> Element 一般表示集合中的元素
    public interface Iterator<E> {
        boolean hasNext();
        
        E next();
        
        default void remove() {
            throw new UnsupportedOperationException("remove");
        }
        
        default void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (hasNext())
                action.accept(next());
        }
    }
    // <T> Type 表示传输参数的类型
    public interface Iterable<T> {
        // 返回一个内部元素为T类型的顺序迭代器
        Iterator<T> iterator();
        
        // 对Iterable中的元素进行指定的操作
        default void forEach(Consumer<? super T> action) {
            Objects.requireNonNull(action);
            for (T t : this) {
                action.accept(t);
            }
        }
        
        // 返回一个内部元素为T类型的并行迭代器
        default Spliterator<T> spliterator() {
            return Spliterators.spliteratorUnknownSize(iterator(), 0);
        }

    }
    ```
##### 迭代器取代了Java Collection Framework中的Enumeration:
        1.迭代器在迭代期间可以从集合中移除元素。
        
        2.方法名得到了改进，Enumeration的方法名称都比较长。
        
##### 为什么一定要实现Iterable接口，为什么不直接实现Iterator接口？
 
    1. 在jdk 1.5以后，引入了Iterable，使用foreach语句（增强型for循环）必须使用Iterable类。
    
    2. Java设计者让Collection继承于Iterable而不是Iterator接口。首先要明确的是，Iterable的子类Collection，Collection的子类List，
    Set等，这些是数据结构或者说放数据的地方。Iterator是定义了迭代逻辑的对象，让迭代逻辑和数据结构分离开来，这样的好处是可以在一种数据结
    构上实现多种迭代逻辑。
    
    3. 更重要的一点是：每一次调用Iterable的Iterator（）方法，都会返回一个从头开始的Iterator对象，各个Iterator对象之间不会相互干扰，
    这样保证了可以同时对一个数据结构进行多个遍历。这是因为每个循环都是用了独立的迭代器Iterator对象。

#####  ConcurrentModificationException异常
    ```
    private class Itr implements Iterator<E> {
            /**
             * 游标
             * Index of element to be returned by subsequent call to next.
             */
            int cursor = 0;
    
            /**
             * 上一个游标，remove时使用
             * Index of element returned by most recent call to next or
             * previous.  Reset to -1 if this element is deleted by a call
             * to remove.
             */
            int lastRet = -1;
    
            /**
             * modCount是当前集合的版本号，修改会+1
             * expectedModCount是迭代器的版本号，初始化为modCount
             * The modCount value that the iterator believes that the backing
             * List should have.  If this expectation is violated, the iterator
             * has detected concurrent modification.
             */
            int expectedModCount = modCount;
    
            public boolean hasNext() {
                return cursor != size();
            }
    
            public E next() {
                checkForComodification();
                try {
                    int i = cursor;
                    E next = get(i);
                    lastRet = i;
                    cursor = i + 1;
                    return next;
                } catch (IndexOutOfBoundsException e) {
                    checkForComodification();
                    throw new NoSuchElementException();
                }
            }
    
            public void remove() {
                if (lastRet < 0)
                    throw new IllegalStateException();
                checkForComodification();
    
                try {
                    AbstractList.this.remove(lastRet);
                    if (lastRet < cursor)
                        cursor--;
                    lastRet = -1;
                    expectedModCount = modCount;
                } catch (IndexOutOfBoundsException e) {
                    throw new ConcurrentModificationException();
                }
            }
    
            final void checkForComodification() {
                if (modCount != expectedModCount)
                    throw new ConcurrentModificationException();
            }
        }
    ```

##### 快速失败机制 fail-fast & 失败安全机制 fail-safe
    https://www.cnblogs.com/thisiswhy/p/12186799.html