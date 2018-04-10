ArrayList & Vector\(实现List Interface -- 且都是数组的实现--查询高效，插入和删除低效\)

unsafe             safe

扩容50%        扩容100%

ArrayList o\(1\) 操作：index-》查询数据； index-》插入数据； 末尾插入或者删除数据,所有操作线程不同步，所以不安全

LinkedList 适合从中间插入或者删除，线程不安全

