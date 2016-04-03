Open Replicator is a high performance MySQL binlog parser written in Java. It unfolds the possibilities that you can parse, filter and broadcast the binlog events in a real time manner.

### note ###
For MySQL 5.6.6 users, binlog\_checksum system variable is NOT supported by open-replicator at the moment, please set it to NONE.

### svn ###
svn checkout http://open-replicator.googlecode.com/svn/trunk/ open-replicator-read-only

### github ###
https://github.com/whitesock/open-replicator

### releases ###
1.0.7
  * release date: 2014-05-12
  * support signed tinyint, smallint, mediumint, int, bigint

1.0.6
  * release date: 2014-05-08
  * remove dependency commons-lang, log4j
  * support MYSQL\_TYPE\_TIMESTAMP2, MYSQL\_TYPE\_DATETIME2, MYSQL\_TYPE\_TIME2

1.0.0
  * release date: 2011-12-29

### maven ###
```
<dependency>
	<groupId>open-replicator</groupId>
	<artifactId>open-replicator</artifactId>
	<version>1.0.7</version>
</dependency>
```

### usage ###
```
final OpenReplicator or = new OpenReplicator();
or.setUser("root");
or.setPassword("123456");
or.setHost("localhost");
or.setPort(3306);
or.setServerId(6789);
or.setBinlogPosition(4);
or.setBinlogFileName("mysql_bin.000001");
or.setBinlogEventListener(new BinlogEventListener() {
    public void onEvents(BinlogEventV4 event) {
        // your code goes here
    }
});
or.start();

System.out.println("press 'q' to stop");
final BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
for(String line = br.readLine(); line != null; line = br.readLine()) {
    if(line.equals("q")) {
        or.stop();
        break;
    }
}
```

### parses ###
BinlogEventParser is plugable. All available implementations are registered by default, but you can register only the parsers you are interested in.

![http://dl.iteye.com/upload/attachment/0070/3054/4274ab64-b6d2-380b-86b2-56afa0de523d.png](http://dl.iteye.com/upload/attachment/0070/3054/4274ab64-b6d2-380b-86b2-56afa0de523d.png)

### senarios ###
**1 Statement based replication**
```
mysql> insert into ta values(null, 'kevin', 100, 99.99);
  QueryEvent[...,databaseName=test,sql=BEGIN]
  IntvarEvent[...,type=2,value=2]
  QueryEvent[...,databaseName=test,sql=insert into ta values(null, 'kevin', 100, 99.99)]
  XidEvent[...,xid=13]
mysql> update ta set grade=100.00 where id = 2;
  QueryEvent[...,databaseName=test,sql=BEGIN]
  QueryEvent[...,databaseName=test,sql=update ta set grade=100.00 where id = 2]
  XidEvent[...,xid=14]
mysql> delete from ta where id = 2;
  QueryEvent[...,databaseName=test,sql=BEGIN]
  QueryEvent[...,databaseName=test,sql=delete from ta where id = 2]
  XidEvent[...,xid=15]
```

**2 Row based replication**
```
mysql> insert into ta values(null, 'kevin', 100, 99.99);
  QueryEvent[...,databaseName=test,sql=BEGIN]
  TableMapEvent[...]
  WriteRowsEvent[...,rows=[Row[columns=[1, kevin, 100, 99.99]]]]
  XidEvent[...,xid=9]
mysql> update ta set grade=100.00 where id = 1;
  QueryEvent[...,databaseName=test,sql=BEGIN]
  TableMapEvent[...]
  UpdateRowsEvent[...,rows=[Pair[before=Row[columns=[1, kevin, 100, 99.99]],after=Row[columns=[1, kevin, 100, 100.00]]]]]
  XidEvent[...,xid=10]
mysql> delete from ta where id = 1;
  QueryEvent[...,databaseName=test,sql=BEGIN]
  TableMapEvent[...]
  DeleteRowsEvent[...,rows=[Row[columns=[1, kevin, 100, 100.00]]]]
  XidEvent[...,xid=11]
```
