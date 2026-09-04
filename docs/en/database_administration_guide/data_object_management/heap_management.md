# Heap Management

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-14T11:00:03.860Z pushedAt=2026-08-21T10:12:05.811Z -->

## Introduction

A heap page is a physical storage structure used by oGRAC to store table data. Data stored in a heap page has no particular order, and rows are allocated on demand during insertion. It is typically used for tables without indexes or for unsorted tables. When there is insufficient space for inserting data into a table, space is requested and extended in the unit of extent.

## Segment-Page Structure

The tablespace hierarchy of oGRAC can be divided into `TableSpace->Segment->Extent->Page->Row`.
A tablespace (`TableSpace`) consists of multiple segments (`Segment`), and common segment types include data segments, index segments, and rollback segments. The next level below `Segment` is `Extent`, followed by `Page`, with the default being `1 extent = 8 pages`. A page is the smallest operation unit for database disk management, and the default page size of oGRAC is `8K`.  
Query the page size of the current database:

```shell
SHOW PARAMETER PAGE_SIZE;
```

## Main Structure of a Heap Page

A `Heap Page` mainly consists of `page_head`, `page_tail`, `itl`, `row_dir` (row directory), and the inserted `row` data.

- `Page_head` mainly stores the page ID, type, size, `SCN`, `table_id`, `user_id`, `freeBegin`, `freeEnd`, and `freeSize`, and manages `dir` and `itl`.

- `ITL` list: manages the transaction information on the current page and is used for transaction isolation judgment. The initial number is 2, and it can be extended as concurrent transactions on a page increase. (Once `itl` is extended on a `page`, its space is not returned to the page space.)

- `row_dir`: the offset address dictionary of row data. One `dir` corresponds to one `row`. After a `row` is deleted, its corresponding `dir` also disappears. The position of a `row` in a `page` can be found through `dir`.

- `freeBegin`, `freeEnd`, and `freeSize` are used together for free space management on the current page.

- `table_id` and `user_id` identify the table and the user. In particular, a page belongs exclusively to one table, and data on a page never spans multiple tables. When page space is allocated to a table, it is allocated by `extent`, meaning that by default 8 pages are allocated to a table on a per operation basis.

## Heap Page Free Space

The heap page free space management mechanism of oGRAC achieves both efficient local utilization and intelligent global allocation, balancing performance, flexibility, and storage efficiency.

### Heap Page Free Space Management

Dynamic awareness of free capacity: oGRAC's heap page manages the free space on a page primarily through three attributes: `freeBegin` indicates the starting position of the free space, `freeEnd` indicates the end of the free space, and `freeSize` indicates the total size of the free space on the current page. Normally, `freeSize >= freeEnd – freeBegin`, because when a row is deleted from a page, only the `is_deleted` field in the `row_header` is set to `TRUE`, and the actual space is not reclaimed. Therefore, free space may still exist before `freeBegin`.

On-demand page compaction: oGRAC uses page compaction (`page compact`) to handle free space. Page compaction is triggered in two cases:
When a record is inserted into the current page, or when a row on the current page is updated but the original `row` space is insufficient after the update, requiring "row migration" to the free space within the page. Assuming the currently required space is `rowSize`, the following stages are involved:

- Determine the relationship between the current page's `freeSize` and `rowSize`. If `freeSize` is insufficient, the row cannot be inserted into this page;

- If `freeSize >= rowSize`, but the space of `freeEnd – freeBegin` is smaller than `rowSize`, page compaction is required. That is, the deleted row space before `freeBegin` on the page is cleaned up, moving `freeBegin` forward so that `freeSize = freeEnd – freeBegin`, and then the new data is inserted at `freeBegin`.

### Heap Page Free Space Scheduling

Space management of a single page is a local optimization, while at the global level of the entire table, intelligent routing among multiple pages is achieved through the Map Tree structure. The `Insert` operation can quickly locate the most suitable heap page based on the size of the data to be inserted, avoiding blind traversal or unbalanced load.

- The free space of a heap page is maintained and managed through `Map Tree`.

- Typically, after a table is created, no heap page is attached to it. When the first statement is inserted, an `extent` (8 heap pages by default) is allocated from the tablespace to the table, where the first page serves as the `segment`, the second page serves as the `map page` to manage heap pages, the `map_node` in the `map page` stores the `page_id` of all heap pages it manages, and the remaining pages are used for storing data.

- In the initial state, a `Map Tree` contains only one map page and 6 heap pages.

- When each map page is initialized, six `list` linked lists are initialized, with each `list` corresponding to a different space range (the range becomes progressively larger). Subsequently, each heap page mounts its corresponding `map_node` in the map page to the corresponding `list` according to its current `freeSize` (the list may be empty). Each `map_node` has a one-to-one correspondence with a heap page; that is, through this `list` linked list, the map page can clearly know the distribution of free space in the heap page it currently manages.

- A `map page` manages at most 1014 heap pages. As the data inserted into the table increases, the number of pages increases. After the threshold is exceeded, a new `map page` is requested for management, at which point the tree height becomes 2. The `map page` also needs to be managed by the `map page` of the `root` node, and at this time the `map_node` stores the `page_id` of the `map page`. The `map page` of the root node is on the `segment page`.

- In this `Map Tree` structure, heap pages appear only at leaf nodes. Only the last `map page` of each level can insert new lower-level nodes for extension. The list of the upper-level `map page` that manages map pages stores the `map_node` of each map page.

### Heap Page Free Page Lookup

When inserting a row into a table, the target `heap page` is determined by its `rowSize` through the `Map Tree`, which finds a `heap page` with a suitable `freeSize`. The process mainly involves the following steps: First, based on `rowSize`, the row is assigned to a specific list, referred to as the `targetList`. Starting from the `root` node, for each `map page`, a `map_node` is randomly selected from a list ranging from `targetList` to `list[5]`, and the corresponding `map page` at the next level is located based on this `map_node`. This process repeats until a specific `heap page` is determined. The row is then inserted into the corresponding `heap page`, and the position of the `map_node` corresponding to that `heap page` within its `list` is adjusted.

## Heap Page Advantages

The `Heap Page` management mechanism in oGRAC provides solid support for high performance, high availability, and scalability of the database at the physical storage layer.

- Efficient write performance: Insert operations do not need to maintain data order; they append directly using the `freeBegin` pointer, achieving fast writes with low complexity. Space can be allocated in batches by extent (8 pages), reducing I/O overhead caused by frequent requests and improving batch import efficiency.

- Flexible space reuse: The page compaction mechanism reclaims the space of deleted rows only when new space is needed, balancing cleanup cost and write latency.

- Intelligent free space scheduling: The `Map Tree` hierarchical structure is introduced to organize multiple `Heap Pages` into different linked lists by `freeSize`. When writing data, the most suitable data page can be quickly matched according to `rowSize`, avoiding high concurrency on a single page caused by full table scans or random distribution, and improving I/O efficiency.

- Good scalability: `Map Tree` supports hierarchical extension, and a single `map page` can manage a massive number of heap pages, enabling a large number of data pages to be managed at a lower hierarchy level.
