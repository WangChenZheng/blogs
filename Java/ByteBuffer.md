浅学一下 java.nio.ByteBuffer

## 一、ByteBuffer类关系图

![图片](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/%E6%88%AA%E5%B1%8F/ByteBuffer.png)

## 二、属性

### Buffer

Buffer缓冲区是特定原始类型(short、byte、int、long、boolean、char、float、double)的元素的线性有限序列。有三个基本属性分别为position, limit, capacity;

+ position指要读取或写入的下一个元素的索引

+ limit指禁止读取或写入的第一个元素的索引

+ capacity指缓冲区可以包含的元素数

![图片](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/%E6%88%AA%E5%B1%8F/Buffer.jpg)

## 三、方法

#### 1. Get方法

读取缓冲区的position位置的字节。对于getInt来说就是读取sizeof(int)的字节数。

#### 2. Put方法

将字节存储至缓冲区的position位置。对于putInt来说就是将sizeof(int)字节的数据存储至position的位置。

#### 3. allocate申请内存方法

+ allocateDirect(int): ByteBuffer

```
public static ByteBuffer allocateDirect(int capacity) {
    // 申请直接内存
    return new DirectByteBuffer(capacity);
}
```

+ allocate(int): ByteBuffer

```
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    
    return new HeapByteBuffer(capacity, capacity);  
}
```

## 四、堆内内存与堆外内存

#### 1. HeapByteBuffer

从**堆内内存**中开辟一块缓冲区。堆内内存完全由JVM管理，JVM有垃圾回收机制，所以我们一般不需要关注对象的内存如何回收。

#### 2. DirectByteBuffer

从**堆外内存**中开辟一块缓冲区。堆外内存直接受操作系统管理，并不受JVM垃圾回收机制影响。

#### 3. 堆内内存的优缺点

+ 堆内内存=新生代+老年代（JDK8）

+ 内存完全由JVM管理

+ 网络I/O、文件读写时性能差：当进行网络I/O或文件读写时，如果使用**堆内内存**，JVM首先会创建一个由系统管理的**堆外内存**然后再去执行真正的读写操作。

#### 3. 堆外内存的优缺点

+ 减少了垃圾回收

+ 加快了网络I/O、文件读写速度

+ 内存难以控制，内存溢出时难以排查。

## 五、堆外内存的申请

```
DirectByteBuffer(int cap) {
    
    super(-1, 0, cap, cap);

    // 计算所需内存大小
    // 是否页对齐
    boolean pa = VM.isDirectMemoryPageAligned();
    // 获取页尺寸
    int ps = Bits.pageSize();
    // 如果页对齐的话，就加上一页的大小
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));

    // 告送内存管理器要分配内存
    Bits.reserveMemory(size, cap);

    // 分配内存的过程
    long base = 0;
    try {
        // 分配内存
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        // 若内存溢出则去除保留内存
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    //初始化内存
    unsafe.setMemory(base, size, (byte) 0);
    // 计算内存地址
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }

    // 创建Cleaner用于内存释放
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

利用反射获取unsafe对象

```
// Unsafe无法直接使用，需要通过反射来获取
private static Unsafe getUnsafe() {
    try {
        Class clazz = Unsafe.class;
        Field field = clazz.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        return (Unsafe) field.get(null);
    } catch (IllegalAccessException | NoSuchFieldException e) {
        throw new RuntimeException(e);
    }
}
```

参考：

[一文搞懂堆外内存（模拟内存泄漏） - 简书](https://www.jianshu.com/p/a63c3ace0a2f)

[bytebuffer gc](https://blog.csdn.net/u013161278/article/details/112632583)
