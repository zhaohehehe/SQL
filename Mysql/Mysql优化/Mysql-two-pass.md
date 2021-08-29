## two-pass
如果排序字段超过max_length_for_sort_data，说明数据比较大，可能需要消耗较大的IO
根据where条件读取行指针和 order by 列到sort buffer或者临时temporary

1. where条件筛选数据。
2. 按行读取数据。
3. 加载数据到sort_buffer
  - `two-pass`存储orderByKey和`行指针`到sort_buffer（第1次IO）
  - `one-pass`存储orderByKey和`select colums`到sort_buffer（第1次IO）
4. 如果缓冲区不满,一次排序即可；如果缓冲区满了，反复每次读取sort_buffer_size范围内的数据行进行内存排序，排序结果写入`temporary${n}`临时文件中。
5. Merge buffer。合并多个临时文件`temporary${n}`。
6. 读取排序结果。
  - `two-pass`根据临时文件中行指针随机读取底层物理存储记录。（第2次IO）
  - `one-pass`直接读取临时文件即可。
