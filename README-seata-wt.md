# 问题处理1

存在大量事务，经确认，trx_mysql_thread_id=0 的事务全部为XA事务。
因为trx_mysql_thread_id=0 的事务无法通过kill trx_mysql_thread_id 的方式处理，所以，需要回滚这些XA事务。

查看XA事务信息

mysql> XA RECOVER;
+----------+--------------+--------------+-----------------------------------------------------+
| formatID | gtrid_length | bqual_length | data                                                |
+----------+--------------+--------------+-----------------------------------------------------+
|     9752 |           33 |           18 | 172.17.0.5:8091:41940156638318592-41940157280047104 |
|     9752 |           33 |           18 | 172.17.0.5:8091:41940156638318592-41940159746297856 |
+----------+--------------+--------------+-----------------------------------------------------+
2 rows in set (0.00 sec)

 XA事务回滚命令的格式：xa rollback 'left(data,gtrid_length)'，'substr(data,gtrid_length+1,bqual_length)', formatID;

 以上查出来的信息拼接结果为（以下举其中一个为例）
 
 mysql> select concat('xa rollback \'',left('172.17.0.5:8091:41940156638318592-41940157280047104',33),'\' , \''     ,substr('172.17.0.5:8091:41940156638318592-41940157280047104',33+1,18),'\' , ',9752) jb ; 
 +-------------------------------------------------------------------------------+
 | jb                                                                            |
 +-------------------------------------------------------------------------------+
 | xa rollback '172.17.0.5:8091:41940156638318592' , '-41940157280047104' , 9752 |
 +-------------------------------------------------------------------------------+
 1 row in set (0.00 sec)
 
mysql> xa rollback '172.17.0.5:8091:41940156638318592' , '-41940157280047104' , 9752;
Query OK, 0 rows affected (0.00 sec)



# XAER_RMFAIL: The command cannot be executed when global transaction is in the  IDLE state

2020-08-27 16:42:35.807  INFO 6983 --- [_RMROLE_1_10_16] Branch Rollbacked result: PhaseTwo_RollbackFailed_Retryable
2020-08-27 16:42:36.724  INFO 6983 --- [_RMROLE_1_11_16] rm handle branch rollback process:xid=172.17.0.5:8091:42287542946516992,branchId=42287545605705728,branchType=XA,resourceId=jdbc:mysql://172.81.203.33:3306/db_seata,applicationData=null
2020-08-27 16:42:36.724  INFO 6983 --- [_RMROLE_1_11_16] Branch Rollbacking: 172.17.0.5:8091:42287542946516992 42287545605705728 jdbc:mysql://172.81.203.33:3306/db_seata
2020-08-27 16:42:36.903  INFO 6983 --- [_RMROLE_1_11_16] 172.17.0.5:8091:42287542946516992-42287545605705728 rollback failed since XAER_RMFAIL: The command cannot be executed when global transaction is in the  IDLE state

叠加了spring事务注解
