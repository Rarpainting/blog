# Golang

<!-- TOC -->

- [Golang](#golang)
	- [Happens Before](#happens-before)
	- [channel](#channel)
	- [go runtime](#go-runtime)
		- [线程实现模型基础](#线程实现模型基础)
		- [go 线程实现模型](#go-线程实现模型)
		- [调度器字段](#调度器字段)
		- [M/P/G 状态转换](#mpg-状态转换)
			- [M/Machine](#mmachine)
			- [P/Processor](#pprocessor)
			- [G/Goroutine](#ggoroutine)
	- [Go Ast](#go-ast)
		- [ast.Ident](#astident)
		- [ast.Object](#astobject)
	- [陷阱](#陷阱)
		- [返回值](#返回值)
		- [defer](#defer)
		- [http 响应](#http-响应)
		- [失败的类型断言](#失败的类型断言)
		- [阻塞的 goroutine 和资源泄漏](#阻塞的-goroutine-和资源泄漏)
		- [使用指针接受方法](#使用指针接受方法)
		- [Reflect](#reflect)
		- [String](#string)
		- [Slice](#slice)
		- [Interface](#interface)
		- [Map](#map)
			- [创建 -- makemap](#创建----makemap)
			- [访问 -- mapaccess](#访问----mapaccess)
			- [分配 -- mapassign](#分配----mapassign)
			- [删除 -- mapdelete](#删除----mapdelete)
			- [扩容 -- growWork](#扩容----growwork)
			- [总结](#总结)
		- [math/rand](#mathrand)
		- [MySQL](#mysql)
			- [`go-sql-driver/mysql`](#go-sql-drivermysql)
			- [github issues](#github-issues)
		- [Decimal](#decimal)

<!-- /TOC -->

## Happens Before

如果要让一个对变量 v 的写操作 w 所产生的结果能够被对该变量的读操作 r 观察到, 那么需要同时满足如下两个条件

- 读操作 r 未先于写操作 w 发生

- 没有其他对此变量的写操作后于写操作 w 且 先于读操作 r 发生

即在一轮读写操作中 其他写操作 / 本次写操作 w / 本次读操作 r
顺序:

其他写操作 -> 本次写操作 w -> 本次读操作 r

## channel

> CSP 思想

![channel](channel-tricks.png)

channel 的 happen before 规则:
- 第 n 个 `send` 一定 `happen before` 第 n 个 `receive finished`, 不管是 buffered channel 还是 unbuffered channel
- 对于 capacity 为 m 的 channel, 第 n 个 `receive` 一定 `happen before` 第 (n+m) `send finished`
- m=0 unbuffered. 第 n 个 `receive` 一定 `happen before` 第 n 个 `send finished`
- channel 的 close 一定 `happen before` receive 端得到通知, 得到通知意味着 receive 收到一个 channel close 后的零值, 而 close 之后的 send 则发生 panic

## go runtime

### 线程实现模型基础

`[M:1]` 用户级线程模型

- 线程的创建, 调度, 同步都由用户的进程户态的线程库处理
- 避免了 户态 和 核态 的切换调度, 处理速度快
- 线程空间仅对应一个内核调度实体, (线程阻塞时)无法调度到其他处理器, 无法通过优先级调度

`[1:1]` 内核级线程模型

- 用户线程对应各自的调度实体
- 能依赖内核的调度能力
- 对性能影响大

`[M:N]` 两级线程模型

- 拥有多个调度实体
- 多个线程对应一个调度实体
- 需要 核态 和 户态 同时参与调度, 设计复杂

### go 线程实现模型

| 中文名称              | 源码中的名称          | 作用域     | 需要说明                             |
| :--:                  | :--:                  | :--:       | :--:                                 |
| 全局 M 列表           | runtime.allm          | 运行时系统 | 能用于存放所有 M 的列表              |
| 全局 P 列表           | runtime.allp          | 运行时系统 | 能用于存放所有 P 的列表              |
| 全局 G 列表           | runtime.allg          | 运行时系统 | 能用于存放所有 G 的列表              |
| 调度器的空闲 M 列表   | runtime ' sched.midle | 调度器     | 被用于存放空闲 M 的列表              |
| 调度器的空闲 P 列表   | runtime ' sched.pidle | 调度器     | 被用于存放空闲 P 的列表              |
| 调度器的可运行 G 队列 | runtime ' sched.runq  | 调度器     | 被用于存放可运行 G 的队列            |
| 调度器的的自由 G 队列 | runtime ' sched.gfree | 调度器     | 被用于存放自由 G 的列表              |
| P 的可运行 G 队列     | runq                  | 本地 P     | 被用于存放当前 P 中的可运行 G 的队列 |
| P 的自由 G 列表       | gfree                 | 本地 P     | 被用于存放当前 P 中的自由 G 的列表   |

### 调度器字段

| 字段名称   | 数据类型 | 用途描述                                                 |
| :--:       | :--:     | :--:                                                     |
| gcwaiting  | uint32   | 作为垃圾回收任务被执行期间的辅助标记, 停止计数和通知机制 |
| stopwait   | int32    |                                                          |
| stopnote   | Note     |                                                          |
| sysmonwait | uint32   | 作为系统检测任务被执行期间的停止计数和通知机制           |
| sysmonnote | Note     |                                                          |

### M/P/G 状态转换

![线程模型](006.png)

#### M/Machine

- 与内核线程 1:1 对应

#### P/Processor

![P 状态转换](008.png)

#### G/Goroutine

![G 状态转换](009.png)

## Go Ast

### ast.Ident

词信息

- .NamePost Token.Post -- 位置
- .Name string -- 名字
- .Obj *Object 具体内容

### ast.Object

- .Kind ObjKind 全局类型
- .Name string 声明的名字
- .Decl interface{} 对应以下字段:
  - XssSpec
  - FuncDecl
  - LabeledStmt
  - AssignStmt
  - Scope
  - nil
- .Data interface{}
  - 特殊对象数据 / nil
- .Type interface{} --

## 陷阱

### 返回值

- 有名返回值 -- func f() (v Type) {} -- 在函数声明时已经被定义
- 匿名返回值 -- func f() Type {} -- 在 return 执行时被声明

结论:
`defer` 语句能访问有名返回值
      不能直接访问匿名返回值

### defer

`defer` 的 颗粒度 是 函数级 的, 即 defer 会在函数结束时调用, 而不在 代码块

### http 响应

resp, err := http.Get("https://api.ipify.io?format=content")

当发生 http 的重定向时, err 和 resp 都**不为空**
因此保险的做法:

```golang
resp, err := http.Get("https://api.ipify.io?format=content")
if resp != nil {
    defer resp.Body.Close()
}
if err != nil {
    fmt.Println("sorry")
    return
}
// do anything...
```

### 失败的类型断言

失败的类型断言 返回断言声明中 **类型** 的 "零" 值

其中, `interface{}`和 `slice` 的零值为 `nil`

### 阻塞的 goroutine 和资源泄漏

例子:

```golang
func First(query string, replicas ...Search) Result {
  c := make(chan Result)
  searchReplicas := func(i int) {c <- replicas[i](query)}
  for i := range replicas {
    go searchReplicas(i)
  }
  return <- c
}
```

该例子导致 chan 阻塞, 从而泄漏大量的 goroutine

解决方法:

1. 申请足够的 chan , 缓存所有的结果

```golang
func First(query string, replicas ...Search) Result {
  c := make(chan Result, len(replicas))
  //...
}
```

2. 添加一个 default 选项

```golang
func First(query string, replicas ...Search) Result {
  //...
  searchReplicas := func(i int) {
    select {
      case c <- replicas[i](query):
      default:
    }
  }
  //...
}
```

3. 添加另一个特殊的 chan 来终止 goroutine

```golang
func First(query string, replicas ...Search) Result {
  //...
  done := make(chan struct())
  defer close(done)
  searchReplicas := func(i int) {
    select {
      case c <- replicas[i](query):
      case <-done:
    }
  }
  //...
}
```

4. (??) 指针构成的 "循环引⽤" 加上 runtime.SetFinalizer 会导致内存泄露

### 使用指针接受方法

### Reflect

**reflect 只能拿非匿名函数/字段, 匿名函数/字段需要靠 runtime 获取**

```golang
// 等同于
// runtime/type.go^_type
type rtype struct {
  size       uintptr
  ptrdata    uintptr  // number of bytes in the type that can contain pointers
  hash       uint32   // hash of type; avoids computation in hash tables
  tflag      tflag    // extra type information flags
  align      uint8    // alignment of variable with this type
  fieldAlign uint8    // alignment of struct field with this type
  kind       uint8    // enumeration for C
  alg        *typeAlg // algorithm table
  gcdata     *byte    // garbage collection data
  str        nameOff  // string form
  ptrToThis  typeOff  // type for pointer to this type, may be zero
}

// 示例:
// *struct
typ: {
  size: 8
  ptrdata: 8
  hash: 474031097
  tflag: 0
  align: 8
  fieldAlign: 8
  kind: 54 // 0x36 -- (1)0x36&0x1f=0x16
  alg: {
    hash: runtime.aeshash64
    equal: runtime.memequal64
  }
  // ?? 和 gc 有关的 ?
  gcdata: {
    1
  }
  // rtype.str 字段的偏移是相对于 { rtype 起始地址所在的区间的
  // 保存 type 信息区块(即 [md.types, md.etypes)）起始地址 }
  str: 6302
  ptrToThis: 0
}
```

### String

`len(string)` 的结果是 []byte 的结果

但是 `range string` 的结果是 []rune 的结果

### Slice

```golang
// maxElems is a lookup table containing the maximum capacity for a slice.
// The index is the size of the slice element.
var maxElems = [...]uintptr{
    ^uintptr(0),
    maxAlloc / 1, maxAlloc / 2, maxAlloc / 3, maxAlloc / 4,
    maxAlloc / 5, maxAlloc / 6, maxAlloc / 7, maxAlloc / 8,
    maxAlloc / 9, maxAlloc / 10, maxAlloc / 11, maxAlloc / 12,
    maxAlloc / 13, maxAlloc / 14, maxAlloc / 15, maxAlloc / 16,
    maxAlloc / 17, maxAlloc / 18, maxAlloc / 19, maxAlloc / 20,
    maxAlloc / 21, maxAlloc / 22, maxAlloc / 23, maxAlloc / 24,
    maxAlloc / 25, maxAlloc / 26, maxAlloc / 27, maxAlloc / 28,
    maxAlloc / 29, maxAlloc / 30, maxAlloc / 31, maxAlloc / 32,
}
```

注意:
1. 根据类型的大小, 算出最多能申请多少个元素
2. 对于扩容:
  1. 如果申请的容量(cap)是老容量(old.cap)的两倍以上`cap>old.cap*2`, 那么就扩成 cap
  2. 否则, 如果老容量 old.cap 小于 1024, 那么就扩成 old.cap x 2
  3. 再否则, newcap 初始为 old.cap, 一直循环 `newcap += newcap/4`, 直到不小于 cap, newcap 就是最终扩成的大小(注意这里还有个溢出保护, 如果溢出了, 那么 newcap=cap)
3. 如果 append 后长度超过了 cap , 那么一定会触发扩容, 以及数据迁移

### Interface

![interface](2835676-ff10962a15ab2676.png)

```golang
// src/runtime/type.go

// 接口类型的定义
type interfacetype struct {
  typ     _type
  pkgpath name
  mhdr    []imethod
}

type imethod struct {
  name nameOff
  ityp typeOff
}
```

```golang
// src/runtime/runtime2.go

// 记录成功对应的接口类型和实际类型
type itab struct {
  inter *interfacetype // point to interface type
  _type *_type // point to concrete type
  hash  uint32 // copy of _type.hash. Used for type switches.
  _     [4]byte
  fun   [1]uintptr // _type 中匹配 inter 的方法的地址的数组 {fun[0]==0 means _type does not implement inter.}
}

type iface struct {
  tab  *itab
  data unsafe.Pointer
}
```

```golang
// src/runtime/iface.go

// Note: change the formula in the mallocgc call in itabAdd if you change these fields.
type itabTableType struct {
  size    uintptr             // length of entries array. Always a power of 2.
  count   uintptr             // current number of filled entries.
  entries [itabInitSize]*itab // really [size] large
}
```

在 get 的时候, 不仅仅会从 itabTalbe 中查找, 还可能会创建插入, itabTable 使用容量超过 75% 还会扩容

```golang
// src/runtime/iface.go

func getitab(inter *interfacetype, typ *_type, canfail bool) *itab {
  // 先从全局的 itabTable 找 inter 和 typ 的配对记录
  t := (*itabTableType)(atomic.Loadp(unsafe.Pointer(&itabTable)))
  if m = t.find(inter, typ); m != nil {}

  // 找不到已有的, 尝试生成新入口
  m = (*itab)(persistentalloc(unsafe.Sizeof(itab{})+uintptr(len(inter.mhdr)-1)*sys.PtrSize, 0, &memstats.other_sys))
  m.init()

finish: // 完成
}

// 尝试匹配 m.inter 和 m._type
// 如果失败, 设置 m.func[0]=0 , 并返回 缺少/匹配失败的方法
func (m *itab) init() string {
  // 获取除了 iface(kindDirectIface) 和 gc(kindGCProg) 外的结构
  switch t.kind & kindMask {
  }

  // 匹配两个方法
  // 由于严格按照字典序匹配, 因此时间复杂度: O(n+m) (type: m, iface: n)
imethods:
}

// 把新生成 m 加入到全局的 (itabTable *itabTableType) 中
func itabAdd(m *itab) {
  // 已经使用了 75% 的申请空间, 需要扩容
  if t.count >= 3*(t.size/4) { // 75% load factor
    t2 := (*itabTableType)(mallocgc((2+2*t.size)*sys.PtrSize, nil, true))
    // ...
  }
  t.add(m)
}

func (t *itabTableType) add(m *itab) {
  h := itabHashFunc(m.inter, m._type) & mask
  for i := uintptr(1); ; i++ {
    p := (**itab)(add(unsafe.Pointer(&t.entries), h*sys.PtrSize))
    m2 := *p
    if m2 == nil {
      // Use atomic write her\e so if a reader sees m, it also
      // sees the correctly initialized fields of m.
      // NoWB is ok because m is not in heap memory.
      // *p = m
      atomic.StorepNoWB(unsafe.Pointer(p), unsafe.Pointer(m))
      t.count++
      return
    }
  }
}

func itabHashFunc(inter *interfacetype, typ *_type) uintptr {
	// compiler has provided some good hash codes for us.
	return uintptr(inter.typ.hash ^ typ.hash)
}
```

### Map

**基于 Golang1.12**

![hmap](2835676-a7c04338cb67866e.png)

**golang 使用的 hash 算法根据硬件选择, 如果 cpu 支持 aes, 那么使用 aes hash, 否则使用 memhash(memhash 参考 xxhash、cityhash 实现)**

把 hash 值映射到 bucket 时, golang 会把 bucket 的数量规整为 2 的次幂, 而有 m=2b, 则 n%m=n&(m-1), **用位运算规避 mod 的昂贵代价**

```golang
// A header for a Go map.
type hmap struct {
  // Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
  // Make sur\e this stays in sync with the compiler's definition.
  count     int // # live cells == size of map.  Must be first (used by len() builtin)
  flags     uint8 // 写冲突标识
  // 装载因子: loadFactor=count/2^B
  // 如果 loadFactor>6.5 , 触发扩容
  B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
  noverflow uint16 // 溢出桶的近似数, 一般计算 h.noverflow++ /approximate number of overflow buckets; see incrnoverflow for details
  hash0     uint32 // hash seed
  buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
  oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
  nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
  extra *mapextra // optional fields
}

// 基桶 (base buckets): hmap 定义的存放预申请的桶的类型
// 溢出桶 (overflow buckets): 存放后续申请的桶的类型

// mapextra holds fields that ar e not present on all maps.
// mapextra 用于存放非指针数据(用于优化存储和访问)
// len(key)>8 , map 会存储数据的 ptr 而不是 实际值
type mapextra struct {
  // If both key and value do not contain pointers and ar\e inline, then we mark bucket
  // type as containing no pointers. This avoids scanning such maps.
  // However, bmap.overflow is a pointer. In order to keep overflow buckets
  // alive, we stor\e pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
  // overflow and oldoverflow ar e only used if key and value do not contain pointers.
  // overflow contains overflow buckets for hmap.buckets.
  // oldoverflow contains overflow buckets for hmap.oldbuckets.
  // The indirection allows to stor\e a pointer to the slice in hiter.
  overflow    *[]*bmap
  oldoverflow *[]*bmap
  // nextOverflow holds a pointer to a free overflow bucket.
  nextOverflow *bmap
}

const (
  // Maximum number of key/value pairs a bucket can hold.
  bucketCntBits = 3
  bucketCnt     = 1 << bucketCntBits // 8
)

// A bucket for a Go map.
// golang 将 hash 中的 key 和 value 分开存放, 操作更繁琐但是优化了对齐
type bmap struct {
  // tophash generally contains the top byte of the hash value
  // for each key in this bucket. If tophash[0] < minTopHash,
  // tophash[0] is a bucket evacuation state instead.
  // 保存每个 hash 结果的顶部字节(8bit)
  // 如果 tophash[0] < minTopHash, 表示 bucket 在迁移过程中
  tophash [bucketCnt]uint8
  // Followed by bucketCnt keys and then bucketCnt values.
  // NOTE: packing all the keys together and then all the values together makes the
  // code a bit mor e complicated than alternating key/value/key/value/... but it allows
  // us to eliminate padding which would be needed for, e.g., map[int64]int8.
  // Followed by an overflow pointer.
}
```

注意:
- (新的) buckets 集绑定在 `hmap.buckets`
- B 的值必须 `h.B<=15`
- `makemap`(??): buckets 带有 `(2^h.B-1)` 个 base bucket(懒申请)
- `tooManyOverflowBuckets`: 最多拥有 `(2^h.B-1)` 个 overflow bucket; `overLoadFactor`: base+overflow 总数必须小于 `max(bucketCnt{8}, loadFactorNum{13}*( 1<<h.B / loadFactorDen{2} ))`
- 对于 bmap:
  - bmap 是数据集的最小颗粒, 结构: `|<- topHash * 8 ->|<- key * 8 ->|<- value * 8 ->|`
  - `[0, minTopHash)` 里是 topHash 的特殊情况
  - topHash 是 `hash(key, h.hash0)` 的高 8 bit, 用来命中 bmap; 当 `topHash<minTopHash`, 则 `topHash+=minTopHash`
  - key 通过 `key&(1<<h.B)` 命中 bucket
  - 当 key/value 符合 `len(key)>128 byte` , bmap 中存储 key/value 的指针
- 对于 overflow bucket:
  - 如果 `h.B>=4` , 就预先分配部分 `overflow bucket`
  - 一个 `overflow bucket` 串中, non-last overflow bucket: `overflow=nil` ; last overflow bucket: `last.setoverflow(t, (*bmap)(buckets))` 设置为 bucket 的首地址表示到了 overflow 尾

key 的计算公式:

| key | hash                                      | topHash                                     | bucket index                                            |
| :-: | :-:                                       | :-:                                         | :-:                                                     |
| key | `hash := alg.hash(key, uintptr(h.hash0))` | `top := uint8(hash >> (sys.PtrSize*8 - 8))` | `bucket := hash & (uintptr(1)<<h.B - 1)`, 即 hash % 2^B |

#### 创建 -- makemap

`make(map[k]v, hint)`

```golang
func makemap(t *maptype, hint int, h *hmap) *hmap {
  // Architectur\e  Name              Maximum Value (exclusive)
  // ---------------------------------------------------------------------
  // amd64         TASK_SIZE_MAX     0x007ffffffff000 (47 bit addresses)
  // arm64         TASK_SIZE_64      0x01000000000000 (48 bit addresses)
  // ppc64{,le}    TASK_SIZE_USER64  0x00400000000000 (46 bit addresses)
  // mips64{,le}   TASK_SIZE64       0x00010000000000 (40 bit addresses)
  // s390x         TASK_SIZE         1<<64 (64 bit addresses)
  mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
  if overflow || mem > maxAlloc {
    hint = 0
  }

  // B = log_2(hint/13) + 1
  for overLoadFactor(hint, B) { B++ }

  // if B == 0, the buckets field is allocated lazily later (in mapassign)
  // 懒操作
  if h.B != 0 {
    h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
  }
}
```

#### 访问 -- mapaccess

```golang
// mapaccess1 returns a pointer to h[key], renturn **zero object**
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
// func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {
// func mapaccessK(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, unsafe.Pointer) {
  // 写保护
  if h.flags&hashWriting != 0 {
    throw("concurrent map writes")
  }

  // hash(key)
  hash := alg.hash(key, uintptr(h.hash0)) // Hash = hash(key, seed)
  // 获得 key 在桶的索引的偏移
  m := bucketMask(h.B) // (1 << h.B -1)
  b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))

  if c := h.oldbuckets; c != nil {
    // 新旧桶不是相同尺寸, 则认为旧桶为新桶的一半
    if !h.sameSizeGrow() {
    }

    // 有旧桶, 则可能使用旧桶查找
    oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
    if !evacuated(oldb) {
      b = oldb
    }
  }

  // 不停寻找
  for ; b != nil; b = b.overflow(t) {
      if alg.equal(key, k) {
        v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
        return v
      }
  }

  // 找不到的 key 会返回 zoreVal[0]
  return unsafe.Pointer(&zeroVal[0])
}
```

#### 分配 -- mapassign

- 与 `mapaccess` 操作相似, 为一个 key 分配槽
- 添加写保护标志 和 扩容操作

1. hash 表如果正在扩容, 并且这次操作的 bucket 还没搬到新 hash 表中, 那么先进行搬迁(扩容细节下面细说)
2. 在 bucket 中寻找 key, 同时记录下第一个空位置, 如果找不到, 那么就在空位置中插入数据; 如果找到了, 那么就更新对应的 value
3. 找不到 key 就看下需不需要扩容, 需要扩容并且没有正在扩容, 那么就进行扩容, 然后回到第一步
4. 找不到 key , 不需要扩容, 但是没有空 slot, 那么就分配一个 overflow bucket 挂在链表结尾, 用新 bucket 的第一个 slot 放存放数据

```golang
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
  // 写保护
  if h.flags&hashWriting != 0 {
    throw("concurrent map writes")
  }

again: // 扩容, k/v 写入的操作

  // 如果需要做扩容, 则开始扩容
  if h.growing() {
    growWork(t, h, bucket)
  }

bucketloop: // 找到合适的 bucket
 for {
    // already have a mapping for key. Update it.
    // 空间已有, 尝试替换
    if t.needkeyupdate() {
      // 需要复写 key
      typedmemmove(t.key, k, key)
    }
    elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
  }

  // Did not find mapping for key. Allocate new cell & add entry.
  // 为新的 k/v allocate 新的空间

  // growing 标志没有设置, 但是太多的 bucket 已经满了
  // 需要返回以扩容
  if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
    hashGrow(t, h)
    goto again
  }

  // 适合的 bucket 都已经满了, 需要生成新的桶
  if inserti == nil {
    // all current buckets ar\e full, allocate a new one.
    newb := h.newoverflow(t, b)
  }

  // stor\e new key/value at insert position
  if t.indirectkey() { ... }
  if t.indirectvalue() { ... }
  typedmemmove(t.key, insertk, key)

done: // 结束 assign 操作
}

func (h *hmap) newoverflow(t *maptype, b *bmap) *bmap {
    if h.extra != nil && h.extra.nextOverflow != nil {
      // overflow = ovf + t.bucketsize - PtrSize // 最后一个 bmap
      if ovf.overflow(t) == nil {
        // 最后一个 bmap(overflow) 没有使用
        // We'r\e not at the end of the preallocated overflow buckets. Bump the pointer.
        h.extra.nextOverflow = (*bmap)(add(unsafe.Pointer(ovf), uintptr(t.bucketsize)))
      } else {
        // 该 bucket 的所有 bmap 都用完了
        // 把最后一个 bmap(overflow) 置空, 方便 incrnoverflow
        ovf.setoverflow(t, nil)
      }
    } else {
      ovf = (*bmap)(newobject(t.bucket))
    }
    // 用于增长 h.noverflow
    h.incrnoverflow()
    // 放置一个空的 overflow bucket 到 h.overflow 的尾部
    h.setoverflow(t, ovf)
}
```

#### 删除 -- mapdelete

```golang
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
  // 扩容
  bucket := hash & bucketMask(h.B)
  if h.growing() {
    growWork(t, h, bucket)
  }

  // 获得 bucket address 和 hash top
  b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
  bOrig := b
  top := tophash(hash)

search:
  for ; b != nil; b = b.overflow(t) {
    for i := uintptr(0); i < bucketCnt; i++ {
      // 获取实际的 k/v 地址
      k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
        if !alg.equal(key, k2) {
        continue
      }
      v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))

      // 如果是 k/v 的指针, 则只删除指针
      // 如果 存储 k/v 的数据, 则需要 memclear 数据
      if t.indirectkey() {
      } else t.key.kind&kindNoPointers == 0 {
        memclrHasPointers(k, t.key.size)
      }

      // 更新当前的 桶
      b.tophash[i] = emptyOne
      // If the bucket now ends in a bunch of `emptyOne` states,
      // change those to `emptyRest` states.
      if i == bucketCnt-1 {
        if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
          // 如果在该桶的 end 位置(bucket[7]), 同时该 overflow bucket 处于 emptyRest 状态 ??
        }
      }

      TODO:

    notLast:
      h.count--
      break search
    }
  }
}
```

#### 扩容 -- growWork

部分概念:

| 变量                | 释义                                                           |
| :-:                 | :-:                                                            |
| `x *bmap`           | 桶 x 表示与在旧桶时相同的位置, 即位于新桶前半段                |
| `y *bmap`           | 桶 y 表示与在旧桶时相同的位置 + 旧桶数组大小, 即位于新桶后半段 |
| `xi int`	          | 桶 x 的 slot 索引                                              |
| `yi int`            | 桶 y 的 slot 索引                                              |
| `xk unsafe.Pointer` | 索引 xi 对应的 key 地址                                        |
| `yk unsafe.Pointer` | 索引 yi 对应的 key 地址                                        |
| `xv unsafe.Pointer` | 索引 xi 对应的 value 地址                                      |
| `yv unsafe.Pointer` | 索引 yi 对应的 value 地址                                      |

注意:
- growWork 会搬迁两个 bucket, 入参的 bucket 和 h.nevacuate(nevacuate 是一个顺序累加的值)

```golang
func growWork(t *maptype, h *hmap, bucket uintptr) {
  // make sur\e we evacuate the oldbucket corr\esponding
  // to the bucket we'r\e about to use
  evacuate(t, h, bucket&h.oldbucketmask())

  // evacuate one mor\e oldbucket to make progress on growing
  if h.growing() {
  	evacuate(t, h, h.nevacuate)
  }
}

// mapassign
//
// if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
//  hashGrow(t, h)
//  goto again // Growing the table invalidates everything, so try again
// }
//
// If we've hit the load factor, get bigger.
// Otherwise, ther\e ar\e too many overflow buckets,
// so keep the same number of buckets and "grow" laterally.
func hashGrow(t *maptype, h *hmap) {
  // a^=B , 取反 B 中为 "1" 的位
  // a&^=B , 取消 B 中为 "1" 的位
	if !overLoadFactor(h.count+1, h.B) {
    bigger = 0
    // map 的 总数过多, 后续只需要 same size grow 就行了
		h.flags |= sameSizeGrow
  }

  oldbuckets := h.buckets
  newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)

  flags := h.flags &^ (iterator | oldIterator)
  if h.flags&iterator != 0 {
    flags |= oldIterator
  }
  // commit the grow (atomic wrt gc)
  h.B += bigger
  h.flags = flags
  h.oldbuckets = oldbuckets
  h.buckets = newbuckets
  h.nevacuate = 0
  h.noverflow = 0
}

func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
  if !evacuated(b) {
    // 用于扩容的两个扩容目的
		var xy [2]evacDst
		x := &xy[0]
		x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
		x.k = add(unsafe.Pointer(x.b), dataOffset) // dataOffset: 64 位对齐
    x.v = add(x.k, bucketCnt*uintptr(t.keysize))
		if !h.sameSizeGrow() {
			// Only calculate y pointers if we'r\e growing bigger.
			// Otherwise GC can see bad pointers.
			y := &xy[1]
			y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
			y.k = add(unsafe.Pointer(y.b), dataOffset)
			y.v = add(y.k, bucketCnt*uintptr(t.keysize))
		}

		if !h.sameSizeGrow() {
    }
  }
}
```

注意:
- 当 `count/bucket>=6.5` 时, 就会进行扩容, `bucket<<=1`

#### 总结

- 通过 topHash 快速试错加快了查找过程
- 检查写保护: `mapaccess`/访问, `mapassign`/分配(赋值), `mapdelete`/删除, `mapiternext`/遍历, `mapclear`/清空
- 设置写保护: `mapassign`/分配(赋值), `mapdelete`/删除, `mapclear`/清空
- 触发扩容(growWork): `mapassign`/分配(赋值), `mapdelete`/删除
- 8键/8值 分别放置减少了 padding 空间

### math/rand

- 全局的 `rand.globalRand` 使用 `lockedSource` 为 source
- 而自建的 `rand.NewSource` 使用 无锁的 `rngSource` 为 source
- 因此, 在 多线程且无互斥需求 的情况下, 随机数生成器尽量使用 `NewSource`(即 `rngSource`)

### MySQL

#### `go-sql-driver/mysql`

- 同一个事务每一个 Query 的结果集用的都是同一个缓存, 必须把 Query 后的结果集清空才能再次执行

- Stmt
  - `DB.Prepare()` 会重新从连接池获取一条新的连接
  - `Tx.Prepare()` 重用 事务(Tx) 的连接

MySQL 8.0 使用 `caching_sha2_password` 作为默认加密插件, `go-sql-driver/mysql` 本身不支持该插件

因此需要在 MySQL 执行以下步骤, 修改 `'root'@'%'` 的加密方式:

```sql
ALTER USER `root`@`%` IDENTIFIED WITH mysql_native_password BY 'password';

-- 以下方法经验证已失效
-- UPDATE mysql.user SET password=PASSWORD("password") where host='%';
```

同时修改 Golang-MySQL-URL , 添加 `allowNativePasswords=true` 参数(BUG? 在 go-sql-driver 的文档中, 该参数是默认开启的, 然而需要明文添加)

可选: 修改 mysql config
```config
[mysqld]
default-authentication-plugin = mysql_native_password
```

#### github issues

- [Add support for sha256_password pluggable authentication #625](https://github.com/go-sql-driver/mysql/issues/625)
- [Unable to connect to MySQL 8 #785](https://github.com/go-sql-driver/mysql/issues/785)

### Decimal

[Go如何精确计算小数](https://imhanjm.com/2017/08/27/go%E5%A6%82%E4%BD%95%E7%B2%BE%E7%A1%AE%E8%AE%A1%E7%AE%97%E5%B0%8F%E6%95%B0-decimal%E7%A0%94%E7%A9%B6/)