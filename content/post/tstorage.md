---
date: "2024-10-07"
title: "tstorage (Time series embedded db)"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
# Table of Contents  
- [Project Info](#project-info)
- [Example](#example)
- [Internal Document](#internal-document)

## Project Info

Embedded [time series database](https://github.com/nakabonne/tstorage) for usage in a Go application.


## Example

Example on how to use the `tstorage` project can be found [here](https://github.com/nanikjava/timeseriesdb)


## Internal Document

Google doc diagram and explanation can be found inside [here](https://docs.google.com/presentation/d/1di6S73kUc1Q65hMpZy8CSnGXTAqrJQBM6bq59qm5tl4/edit?usp=sharing).

The document contains the following information:

* Structure of `tstorage.memoryPartition` which is used as container to collect metric information in memory

![](/static/media/tstorage/high_level_memory_partition.png)

* Breakdown of data stored inside `tstorage.memoryPartition` in terms of metrics.

![](/static/media/tstorage/tstorage_memory_partition.png)

* Diagram outlining the structure of `tstorage.storage` showing the linked list information of all parititions (memory and disk).

![](/static/media/tstorage/tstorage_storage.png)

* Steps breakdown of when a new partition is created.

![](/static/media/tstorage/new_partition_flow.png)

* Splitting of reading partition file from disk into memory mapped and `meta.json` into `meta` structure.

![](/static/media/tstorage/mapping_files.png)

* Internal representation of the [Gorilla](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf) compression

![](/static/media/tstorage/gorilla_compression.png)
