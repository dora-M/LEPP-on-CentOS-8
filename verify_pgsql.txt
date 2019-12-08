su - postgres
createdb testdb
psql
	\list
	\connect testdb
	CREATE TABLE test_table( col1 int, col2 varchar(10));
	INSERT INTO test_table VALUES (11, 'val1');
	INSERT INTO test_table VALUES (22, 'val2');
	SELECT * FROM test_table;
	CREATE USER testuser WITH PASSWORD 'password';
	GRANT SELECT ON TABLE test_table TO testuser;
	\q
exit

useradd -M testuser
passwd testuser
su - testuser
psql testdb
	select * from test_table;
	\q

exit
getsebool -a | grep httpd_can_network_connect_db
setsebool -P httpd_can_network_connect_db on

vi /var/lib/pgsql/data/pg_hba.conf
	# IPv6 local connections:
	#host    all             all             ::1/128                 ident
	host	testdb		testuser	::1/128			password


systemctl restart postgresql.service
vi /usr/share/nginx/html/pgsql.php
elinks http://localhost/pgsql.php







[root@localhost ~]# su - postgres
[postgres@localhost ~]$ createdb testdb
[postgres@localhost ~]$ psql
psql (10.6)
Type "help" for help.

postgres=# \list
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(4 rows)

postgres=# \connect testdb
You are now connected to database "testdb" as user "postgres".
testdb=# CREATE TABLE test_table( col1 int, col2 varchar(10));
CREATE TABLE
testdb=# INSERT INTO test_table VALUES (11, 'val1');
INSERT 0 1
testdb=# INSERT INTO test_table VALUES (22, 'val2');
INSERT 0 1
testdb=# SELECT * FROM test_table;
 col1 | col2 
------+------
   11 | val1
   22 | val2
(2 rows)

testdb=# CREATE USER testuser WITH PASSWORD 'password';
CREATE ROLE
testdb=# GRANT SELECT ON TABLE test_table TO testuser;
GRANT
testdb=# \q
[postgres@localhost ~]$ exit
logout
[root@localhost ~]# useradd -M testuser
[root@localhost ~]# passwd testuser
[root@localhost ~]# su - testuser
su: warning: cannot change directory to /home/testuser: No such file or directory
[testuser@localhost root]$ psql testdb
could not change directory to "/root": Permission denied
psql (10.6)
Type "help" for help.

testdb=> select * from test_table;
 col1 | col2 
------+------
   11 | val1
   22 | val2
(2 rows)

testdb=> \q
could not save history to file "/home/testuser/.psql_history": No such file or directory
[testuser@localhost root]$ exit
logout
[root@localhost ~]# getsebool -a | grep httpd_can_network_connect_db
httpd_can_network_connect_db --> off
[root@localhost ~]# setsebool -P httpd_can_network_connect_db on
[root@localhost ~]# vi /var/lib/pgsql/data/pg_hba.conf
	# IPv6 local connections:
	#host    all             all             ::1/128                 ident
	host	testdb		testuser	::1/128			password


[root@localhost ~]# systemctl restart postgresql.service
[root@localhost ~]# vi /usr/share/nginx/html/pgsql.php
[root@localhost ~]# cat /usr/share/nginx/html/pgsql.php

<?php
$connStr = "host=localhost port=5432 dbname=testdb user=testuser password=password"
	or die("Could not connect");
echo "Connected successfully to psql";
//simple check
$conn = pg_connect($connStr);
$result = pg_query($conn, "select * from test_table");
if (!$result) {
  echo "An error occurred.\n";
  exit;
}
while ($row = pg_fetch_row($result)) {
  echo "<br />\n";
  echo "col1: $row[0]  col2: $row[1]";
}
echo pg_last_error($conn);
pg_close($conn);
?>
[root@localhost ~]# elinks http://localhost/pgsql.php

                                                     http://localhost/pgsql.php 
   Connected successfully to psql                                               
   col1: 11 col2: val1                                                          
   col1: 22 col2: val2  

