 存储引擎：

#### BigTable：

<img src="picture\image-20211211120513169.png" alt="image-20211211120513169" style="zoom: 50%;" />







#### LSM tree

<img src="picture\image-20211208205001108.png" alt="image-20211208205001108" style="zoom: 80%;" />

可插件化的存储引擎，RocksDB(LSMTree)、FasterKV(Hash)

#### 代码分析

##### db:

​	Write:
​		批量写。最大长度： max_size = size + (128 << 10);
​		从deque中取出一批写操作，进行处理。



##### WriteBatch : 写操作集合

````
The `WriteBatch` holds a sequence of edits to be made to the database.

数据存储格式:|seq|type|key_size|key|value_size|value|
````

##### InternalKey

```
user_key|seq(7byte)|type(1byte)
```

	Get:
		快照读：

````	
|internal_key_size = key_size + 8 (int32变长)|  user key|type 1byte |seq 7 byte|val_size (int32变长)|value
多申请的8个字节用来存储type和SequenceNumber      |      internal_key             |   

````

​	

##### LookUpKey

````
user_key_size|key|seq + kValueTypeForSeek 8byte
var int32
````

​		



##### LogWriter:

​	文件映射内存:https://blog.csdn.net/mg0832058/article/details/5890688
​	写入日志结构block：

````
| crc (4byte)|length(2byte)|type(1byte)|record|
````

##### SkipList

````
线程安全的跳跃表
````

##### MemTable:

​	使用跳表实现。
​	

##### SSTable（ Sorted String Table ）:

###### TableBuilder

​	构造SStable

SSTable文件存储格式：

```
<beginning_of_file>
[data block 1]
[data block 2]
...
[data block N]
[meta block 1] filter block 用来存储过滤器信息，比如布隆过滤，key过滤。
...
[meta block K]
[metaindex block]  Add mapping from "filter.Name" to location of filter data
[index block]  [前缀: block offset size] key前缀 ：小于key前缀的key对应的block的offset以及size
[Footer]        (fixed size; starts at file_size - sizeof(Footer))
<end_of_file>
```

**BlockBuilder**对key的存储是前缀压缩的，对于有序的字符串来讲，这能极大的减少存储空间。但是却增加了查找的时间复杂度，为了兼顾查找效率，每隔K个key，leveldb就不使用前缀压缩，而是存储整个key，这就是**重启点**（restartpoint）。

###### Block结构：

````
|shared size (varint32)|non shared size (varint32)|value size (varint32)|key.data() + shared|value|
|shared size (varint32)|non shared size (varint32)|value size (varint32)|key.data() + shared|value|
|shared size (varint32)|non shared size (varint32)|value size (varint32)|key.data() + shared|value|
|shared size (varint32)|non shared size (varint32)|value size (varint32)|key.data() + shared|value|
restartpoint (int32 offset)|restartpoint (int32)|restartpoint (int32)...|restartpoint num(int32)|
````



Footer:

````
metaindex offset (Varint64)|  metaindex size (Varint64) | index offset (Varint64)| index size (Varint64) | kTableMagicNumber int64

````

##### TableCache:

​	以文件序号作为key的文件缓存。

##### Version:

###### ref:引用计数

###### LevelFileNumIterator

level > 0 层的文件访问迭代器。

###### AddIterators

```
AddIterators(const ReadOptions& options,std::vector<Iterator*>* iters);
将所有层的遍历迭代器，添加到iters中。
	level 0  : table cache iter.
	level >0 : ConcatenatingIterator :TwoLevelIterator 懒加载：只有在访问到某个sst文件时才读取文件创建文件访问迭代器。
```

###### Get

###### UpdateStats

````
f->allowed_seeks--

如果f->allowed_seeks <= 0 && file_to_compact_ == nullptr，更新需要compact的文件以及level。
````

###### RecordReadSample

````c++
声明：
    bool RecordReadSample(Slice key);
	记录样本key，在读取 config::kReadBytesPeriod 个字节后进行样本采集。
	返回true代表需要触发compaction。
        
    查找一个key，需要查找多个sst文件的话，需要进行合并.
调用ForEachOverlapping实现。

````

###### ForEachOverlapping

````c++
ForEachOverlapping(Slice user_key, Slice internal_key, void* arg,bool (*func)(void*, int, FileMetaData*))
遍历每层和key有重合的文件，并执行回调函数func.

````

###### GetOverlappingInputs

````
获取某层所有与[begin,end]重合的文件。
	Level-0 files may overlap each other.
	0层的文件互相之间也可能是重合的，所以如果level=0，需要修改重合区间范围重新搜索。
````

###### OverlapInLevel

````
return true 代表在某层有key重合。
调用SomeFileOverlapsRange 实现。
````

###### SomeFileOverlapsRange

````c++
声明：
    bool SomeFileOverlapsRange(const InternalKeyComparator& icmp,
                           bool disjoint_sorted_files,
                           const std::vector<FileMetaData*>& files,
                           const Slice* smallest_user_key,
                           const Slice* largest_user_key);
在某层的文件中找是否有和smallest_user_key,largest_user_key区间内有重合key的文件。
第一层逐个检查所有文件。
level > 0，使用二分查找。
````

###### PickLevelForMemTableOutput


````c++
返回可以是Memtable compaction的level。
	找到有重合的level
	
    while (level < config::kMaxMemCompactLevel) {
      if (OverlapInLevel(level + 1, &smallest_user_key, &largest_user_key)) {
        break;
      }
      if (level + 2 < config::kNumLevels) {
        // Check that file does not overlap too many grandparent bytes.
        GetOverlappingInputs(level + 2, &start, &limit, &overlaps);
        const int64_t sum = TotalFileSize(overlaps);
        if (sum > MaxGrandParentOverlapBytes(vset_->options_)) {
          break;
        }
      }
      level++;
    }
````

##### VersionEdit



##### VersionSet

<img src="picture\versionset.png" alt="image-20211211120513169" style="zoom: 50%;" />

###### VersionSet::Builder

````c++
VersionSet* vset_;
Version* base_;
LevelState levels_[config::kNumLevels];

void Apply(VersionEdit* edit) 
    // We arrange to automatically compact this file after
    // a certain number of seeks.  Let's assume:
    //   (1) One seek costs 10ms 一次seek花费10ms
    //   (2) Writing or reading 1MB costs 10ms (100MB/s)  读写1MB花费10ms 
    //   (3) A compaction of 1MB does 25MB of IO:  合并1MB需要25MB的IO
    //         1MB read from this level   当前层读取1MB
    //         10-12MB read from next level (boundaries may be misaligned)  读取下一层10-12MB
    //         10-12MB written to next level   写下一层10-12MB
    // This implies that 25 seeks cost the same as the compaction  这意味着 25次seeks == 1MB 数据合并，一次seek大于等于40kb数据合并
    // of 1MB of data.  I.e., one seek costs approximately the     我们有点保守在出发合并之前，允许大约16kb数据1次seek。
    // same as the compaction of 40KB of data.  We are a little
    // conservative and allow approximately one seek for every 16KB
    // of data before triggering a compaction.
void SaveTo(Version* v)
    把当前状态写到v中
````

###### LogAndApply

````c++
Status LogAndApply(VersionEdit* edit, port::Mutex* mu);
1、写入edit到version set中，生成一个新的版本
2、计算下一次合并的最优level
3、写快照文件 
````

###### Finalize

````c++
void Finalize(Version* v);
计算用于下一次合并最优的level。
    level == 0 :
		level 0 文件个数 / 4
	  // We treat level-0 specially by bounding the number of files
      // instead of number of bytes for two reasons:
      // 对于有较大的写缓冲区，最好不要进行太多的合并。
      // (1) With larger write-buffer sizes, it is nice not to do too many level-0 compactions.
      //  
      // (2) The files in level-0 are merged on every read and therefore we wish to avoid too many files when the individual 个别
      // file size is small (perhaps because of a small write-buffer setting, or very high compression ratios, or lots of
      // overwrites/deletions).
	其他层:
		计算这层的文件大小占这层配置的最大空间的比例
        
````

###### Recover

从快照中恢复

###### CompactRange

```

```

###### *PickCompaction

```
获取到需要合并的层级以及文件：
 	size_compaction 	
        compaction score > 1
        如果合并点为空，使用第一个文件进行合并。
        不空，从上次的合并点开始合并。
        没有大于合并点的sst文件，则从第一个文件开始合并。
    seek_compaction:
        文件多次访问但是并没有读到key，合并这个文件。
        如果文件处于第0层，需要遍历找到重合的所有文件。

SetupOtherInputs：
	找到下一层与需要合并的文件。
	优化点：
		找到level以及level+1层最小以及最大值，查找level层与all_start,all_limit重合的文件，记为expand0，
		expand0的文件数大于之前的文件数，且在level+1层需要合并的文件数不变的情况下，扩展level需要合并的文件。
		
	设置合并完的这一层的合并点（compact pointer）
```

###### AddBoundaryInputs

````
AddBoundaryInputs
	因为seq是增加的，所以在同一层当userkey相等时，internalkey会大于
// If there are two blocks, b1=(l1, u1) and b2=(l2, u2) and
// user_key(u1) = user_key(l2), and if we compact b1 but not b2 then a
// subsequent get operation will yield an incorrect result because it will
// return the record from b2 in level i rather than from b1 because it searches
// level by level for records matching the supplied user key.
 查找处于边界的文件
````

##### Compaction



##### DBImpl

###### CompactMemTable

```c++
压缩imm MemTable
调用 WriteLevel0Table,将immtable写入第0层。
Status WriteLevel0Table(MemTable* mem, VersionEdit* edit, Version* base);

```

###### DoCompactionWork

````
````
