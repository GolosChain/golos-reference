Storage Engine
--------------

Being a database Golos Blockchain is required to have a convenient storage engine. Right now this is a [Chainbase](https://github.com/GolosChain/chainbase).


### ChainBase 

  ChainBase is designed to meet the demanding requirments of blockchain applications, but is suitable for use
  in any application that requires a robust transactional database with the ability have near-infinate levels of undo
  history.

  While chainbase was designed for blockchain applications, it is suitable for any program that needs to
  persist complex application state with the ability to undo.

### Features 

  - Supports multiple objects (tables) with multiple indicies (based upon boost::multi_index_container)
  - State is persistant and sharable among multiple processes 
  - Nested Transactional Writes with ability to undo changes

### Dependencies 
  
  - c++11 
  - [Boost](http://www.boost.org/) 
  - CMake Build Process
  - Supports Linux, Mac OS X  (no Windows Support)

### Example Usage 

``` c++
enum tables {
   book_table
};

/**
 * Defines a "table" for storing books. This table is assigned a 
 * globally unique ID (book_table) and must inherit from chainbase::object<> which
 * decorates the book type by defining "id_type" and "type_id" 
 */
struct book : public chainbase::object<book_table, book> {

   /** defines a default constructor for types that don't have
     * members requiring dynamic memory allocation.
     */
   CHAINBASE_DEFAULT_CONSTRUCTOR( book )
   
   id_type          id; ///< this manditory member is a primary key
   int pages        = 0;
   int publish_date = 0;
};

struct by_id;
struct by_pages;
struct by_date;

/**
 * This is a relatively standard boost multi_index_container definition that has three 
 * requirements to be used withn a chainbase database:
 *   - it must use chainbase::allocator<T> 
 *   - the first index must be on the primary key (id) and must be unique (hashed or ordered)
 */
typedef multi_index_container<
  book,
  indexed_by<
     ordered_unique< tag<by_id>, member<book,book::id_type,&book::id> >, ///< required 
     ordered_non_unique< tag<by_pages>, BOOST_MULTI_INDEX_MEMBER(book,int,pages) >,
     ordered_non_unique< tag<by_date>, BOOST_MULTI_INDEX_MEMBER(book,int,publish_date) >
  >,
  chainbase::allocator<book> ///< required for use with chainbase::database
> book_index;

/**
    This simple program will open database_dir and add two new books every time
    it is run and then print out all of the books in the database.
 */
int main( int argc, char** argv ) {
   chainbase::database db;
   db.open( "database_dir", database::read_write, 1024*1024*8 ); /// open or create a database with 8MB capacity
   db.add_index< book_index >(); /// open or create the book_index 


   const auto& book_idx = db.get_index<book_index>().indicies();

   /**
      Returns a const reference to the book, this pointer will remain
      valid until the book is removed from the database.
    */
   const auto& new_book300 = db.create<book>( [&]( book& b ) {
       b.pages = 300+book_idx.size();
   } );
   const auto& new_book400 = db.create<book>( [&]( book& b ) {
       b.pages = 300+book_idx.size();
   } );

   /**
      You modify a book by passing in a lambda that receives a
      non-const reference to the book you wish to modify. 
   */
   db.modify( new_book300, [&]( book& b ) {
      b.pages++;
   });

   for( const auto& b : book_idx ) {
      std::cout << b.pages << "\n";
   }

   auto itr = book_idx.get<by_pages>().lower_bound( 100 );
   if( itr != book_idx.get<by_pages>().end() ) {
      std::cout << itr->pages;
   }

   db.remove( new_book400 );
   
   return 0;
}

```

### Concurrent Access 

By default ChainBase provides no synchronization and has the same concurrency restrictions as any 
boost::multi_index_container.  This means that two or more threads may read the database at the
same time, but all writes must be protected by a mutex.  

Multiple processes may open the same database if care is taken to use interpocess locking on the
database.  

### Persistance 

By default data is only flushed to disk upon request or when the program exits. So long as the program
does not crash in the middle of a call to db.modify(), or db.create() the content of the
database should remain in a consistant state. This means that you should minimize the complexity of the
lambdas used to create and/or modify state.

If the operating system crashes or the computer loses power, then the database will be left in an undefined
state depending upon which memory pages that operating system was able to sync to disk.

ChainBase was designed to be used with blockchain applications where an append-only log of blocks is used
to secure state in the event of power loss. This block log can be replayed to regenerate the full database
state. Dealing with OS crashes, loss of power, and logs, is beyond the scope of ChainBase.

### Portability 

The contents of the database file is dependent upon the memory layout of the computer and process that created
the database. Moving the database to a machine that uses a different compiler, operating system, libraries, or
build type (release vs debug) will result in undefined behavior.  

If portability is desired, the developer will have to export the database to a suitable format. 

### Background 

Blockchain applications depend upon a high performance database capable of millions of read/write 
operations per second.  Additionally blockchains operate on the basis of "eventually consistant" which
means that any changes made to the database are potentially reversible for an unknown amount of time depending
upon the consenus protocol used. 

Existing database such as [libbitcoin Database](https://github.com/libbitcoin/libbitcoin-database) achieve high
peformance using similar techniques (memory mapped files), but they are heavily specialised and do not implement
the logic necessary for multiple indicies or undo history. 

Databases such as LevelDB provide a simple Key/Value database, but suffer from poor performance relative to memory mapped file implementations.

### Design Issues

Anyway being designed as a shared memory-based storage, this causes ChainBase to perform a lot of Disk-I/Os. Generally, Input/Output operations are well known to be one of the slowest types of operation, and usually the main bottleneck in modern systems. 

Since sync and replay processes heavily rely on I/O, it is some decrease in performance is expected; meaning an increase in the time needed to fully perform those same tasks.

Although to a lesser degree, these operations keep going during the normal "production" phase too. For this very reason, concurrently with high I/O spikes, golosd may be slow to respond. This seems to be the cause that leads the node to miss random blocks.

#### Systems on HDD
Specifically, on systems that rely on common Hard Disk for storage (magnetic, non-SSD), both sync and replay can easily take days, literally.
These Drives have pretty low Read/Write speed and IOPS performance - Input/Output (operation) Per Second - that can easily slow down the processes that rely on them.

Steemit, [suggested](https://steemit.com/steem/@steemitblog/steem-0-16-0-official-release) some "Platform Configuration Optimizations", mainly addressed to Linux users. These optimizations are:

    (1) --flush = 100000
    (2) echo    75 | sudo tee /proc/sys/vm/dirty_background_ratio 
        echo  1000 | sudo tee /proc/sys/vm/dirty_expire_centisecs
        echo    80 | sudo tee /proc/sys/vm/dirty_ratio
        echo 30000 | sudo tee /proc/sys/vm/dirty_writeback_centisecs
        
(1) `--flush = 100000` - Is a golosd parameter which flushes (msync - synchronize a file with a memory map) the entire memory-mapped file to disk approximately once every n blocks. So about every 100k blocks for the default/recommended value.

Running some tests, witnesses came to the conclusion that flush doesn't really do much. Actually, we couldn't see any correlation between the flushing time frame and I/O spikes, so we could say that flush seems to do nothing at all really.
For this reason, I would not bother with it, and I would simply set it to 0.

(2) - These are commands that change kernel settings in an attempt to better tune the virtual memory subsystem, specifically for golosd.

`dirty_background_ratio` - Represents the percentage of system memory that once dirty, the kernel can start write to disk. This is a background process doing asynchronous writes, a non-blocking event.

`dirty_ratio` - Represent the percentage of system memory that once dirty, the process doing the writes would start flushing to disk. At this point, all new I/O blocks until dirty pages have been written to disk. This is the process/application itself doing synchronous writes, a blocking event.

`dirty_expire_centisecs` - Denote how long data can be in cache before they have to be written to disk.

`dirty_writeback_centisecs` - Denote how often the kernel threads responsible for writing the dirty pages to disk (pdflush), will wake up to check if there is work to do.

For performance reasons, data is usually not always written out to disk but is temporarily stored in cache instead (dirty pages).
Every `dirty_writeback_centisecs` pdflush wakes up and write to disk (for real) those dirty pages that have been in memory longer than `dirty_expire_centisecs`.
If, due to high I/O, dirty pages keep growing and hit `dirty_background_ratio`, the kernel will start writing to disk regardless of the above parameters (in background, asynchronously).
If in spite of the kernel background flushing, dirty pages hit `dirty_ratio`, the application doing the writes (and so the one generating high I/O causing the dirty pages to grow and hit the limit) will block, pausing all I/O until dirty pages return again below `dirty_ratio` value.

The intent of these changes is to hold the kernel and prevent him from writing cached data to disk too often. In fact, compared to the default settings, these changes will allow more data/dirty pages to remain in memory before the flushing process will kick in. Your disk drive will have to handle bigger writes but much less frequently than before.

But these changes are not the only required. Let us consider consider the sync and replay tasks:

* Hardware will still dictate your overall performance, so there is no any reason to expect some miracles. For this reason, every particular HDD performance is still a limit.

* Anyway, from some tests, it seems that these new settings can help reducing the time needed to perform sync and replay (again, do not expect miracles) but you will need more than 8GB of RAM to actually see some improvements. If your system only has 8 or less GB of RAM, you probably would not see any meaningful difference, if at all.

Considering production mode, long term usage showed that recommended changes do not fit well when golosd enters its usual execution mode.

If during sync and replay we can accept and even profit by occasional slow down/freezes due to big data being written to disk during the flush (instead of having lot of flushes with fewer data do write), This same behavior would increase the chance of missing random blocks once sync/replay is completed.

Allowing more dirty pages to be kept in memory, means that once they need to be written out, the disk will have to manage more data and will need more time to do so. If in the meantime golosd needed to write or read some data from disk to be able to generate a block, it would find the disk too busy to satisfy the requests in time. This would lead to failure in generating the block in the allowed slot time, and so to a shining new missed block.

The scenario in which the disk have more frequent I/O but with fewer data to handle, would be better for the production mode. The I/O would complete faster and the overall disk load would be distributed more fairly over time.

Therefore the suggestion is to lower at least the `dirty_writeback_centisecs` to a value between 500 and 3000 (respectively 5 and 30 seconds).
Another option would be simply switching back to the default kernel settings.

#### Systems on SSD

SSDs have way higher IOPS compared to HDDs. This allows them to satisfy even more I/O requests easily and faster than what an HDD could do.

If it is true that the new version brings more I/O load, and that it may be high enough to cause some issue on HDD, it is also true that that same load should be managed by a common SSD without too much effort, for the reason stated above.

This means that if your system is backed by a Solid State Drive, you probably did not experience all these I/O issues, and there is probably no reason to start bother yourself with VM kernel settings and such at all.

#### Shared Memory - /run/shm
Another option is to try to ditch Disk I/O issue entirely by storing the memory mapped file directly into RAM. At the point of writing, it is my opinion that this is the best way and could probably be a best practice to run golosd.

I find this method funny because pre-v0.16.0 and so pre-chainbase, golosd used to store the blockchain-state in RAM. With v0.16.0 and the introduction of chainbase, devs decided (for good reasons) to change the architecture and store the blockchain-state to disk instead, through a memory mapped file.

The suggestion is to take that file and put it back on RAM. This is easy to do, thanks to the `--shared-file-dir parameter` that allows you to specify the directory to use for the mapped files.

##### How To
For a witness node built as low_memory_node and with only the witness plugin activated, it is recommended to have:

At least 8GB of RAM
At least 4GB of Swap
Check the size of `/run/shm` with `df -h /run/shm`
(by default, it should be equal to 50% of your RAM)

You probably need to mount it, and if its size is smaller than 12G, you should resize it too:
`sudo mount -o remount,size=12G /run/shm`

At this point, you should be ready to go. You can start golosd with:
`./golosd --shared-file-dir /run/shm`

The first time you run golosd with the "/run/shm" method, you will need to:

Copy shared_memory.bin and shared_memory.meta files from your data folder to `/run/shm`
(default data folder: `witness_node_data_dir/blockchain`)
or else

Start golosd forcing a replay that will rebuild the mapped files from your blockchain data (block_log file): `./golosd --shared-file-dir /run/shm --replay`
