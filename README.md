#Patches

---
###Percona-Server
Percona Server 的 Patch 主要是基于 percona server 5.5.23

#####Percona-Server / change_columns_of_statistics_from_int_to_bigint.patch
将 information_schema.user_statistics, information_schema.client_statistics 等统计表中部分 int 型字段改为 bigint ，以免实例长时间运行后其值超过 int 型范围。将 cputime 单位由秒改为微秒，以进行精确统计。

#####Percona-Server / fix_bug_of_max_user_connections.patch
修复实例连接数超过 max_connections 时，连接失败的连接未在 max_user_connection 计数器上减 1 的 BUG。

#####Percona-Server / log_microtime_in_mysql.slow_log.patch
当 log_output = TABLE ，慢查询日志记录在 mysql.slow_log 时，将 query_time 和 lock_time 的单位由秒改为微秒，类型可为 double 或 bigint

#####Percona-Server / make_log_dir_when_install_db.patch
修改 mysql_install_db 脚本，在初始化实例时创建 binlog, relaylog 和 slowlog 文件的目录。

---
### MongoDB

#####mongodb-2.0 / change_the_size_of_first_data_file_to_4MB_when_smallfiles.patch
当设置 smallfiles 选项时，数据文件的初始大小由 4M 开始。当 Database 数量特别大，且 Database 中数据比较少时，可以大大节省磁盘空间。

#####mongodb-2.0 / check_pdfile_version_only_when_forceRepai.patch
MongoDB 每次启动时都会打开每个 database，检查 database 的 pdfile，如果 pdfile 版本不符便 repairDatabase，当 database 数量非常大时，这个检查会非常浪费时间。但是如果 MongoDB 版本没有升级，且 MongoDB 没有异常退出，这个是完全没有必要的，因此将此检查从 MongoDB 的正常启动中去除。

#####mongodb-2.0 / choose_the_closest_secondary_node_to_sync.patch
Replica Set 中加入一个新节点时，新节点会 ping 所有已存在节点，挑选一个与它最近的节点进行初始同步，但如果挑中了 primary 节点，则会对线上业务产生严重影响，此 patch 便新节点首先挑选 secondary 节点进行初始同步，仅当没有可用的 secondary 节点时，才使用 primary 节点。

#####mongodb-2.0 / do_not_get_dbsize_when_listdb.patch
listDatabase 操作会查询所有 database 的数据大小一同返回，相比仅仅列出 database 列表，会多花大量的时间，对线上业务也会产生一定影响。而 MongoDB 内部所有 listDatabase 操作都不需要 database 大小，因此添加一个 withSize 参数，仅当 withSize 参数为 true 时才返回 database 大小。