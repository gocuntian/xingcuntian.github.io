#添加可能需要的包

	sudo apt-get -y install libpcre3-dev build-essential zlib1g-dev libpam0g-dev libssl-dev libperl-dev libxml2-dev gettext libncurses5-dev flex bison gawk cmake

#MySQL安装
	# 修正为以下的语句, 设置mysql无home目录, 无登录权限
	sudo /usr/sbin/groupadd mysql
	sudo /usr/sbin/useradd -r -g mysql -s /bin/false -M mysql

#创建MySQL数据库存放目录
	
	sudo mkdir -p /usr/local/mysql/
	
	sudo mkdir -p /usr/local/mysql/data
	
	sudo chmod +w /usr/local/mysql/data
	
	sudo chown -R mysql:mysql /usr/local/mysql

	wget http://mirrors.sohu.com/mysql/MySQL-8.0/mysql-boost-8.0.0-dmr.tar.gz
	
	tar xf mysql-boost-8.0.0-dmr.tar.gz
	
	cd mysql-8.0.0-dmr/
	
	sudo cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_EXAMPLE_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DENABLE_DTRACE=0 -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EMBEDDED_SERVER=1 -DWITH_BOOST=./boost
	
	sudo make -j `grep processor /proc/cpuinfo | wc -l`
	
	sudo make install -j `grep processor /proc/cpuinfo | wc -l`
	
	sudo ln -s  /web/mysqldata /usr/local/mysql/data
	
	sudo chmod +w /usr/local/mysql
	
	sudo chown -R mysql:mysql /usr/local/mysql
	
	sudo chmod +w /usr/local/mysql/data
	
	sudo chown -R mysql:mysql /usr/local/mysql/data
	
	sudo /sbin/ldconfig

#创建my.cnf配置文件

	sudo rm -fr /etc/my.cnf
	
	sudo vi /etc/my.cnf
	
	[client]
	
	port = 3306
	
	socket = /tmp/mysql.sock
	
	default-character-set = utf8mb4

	[mysql]
	
	prompt= \u@\h [\d]>\_
	no-auto-rehash

	[mysqld]
	port = 3306
	socket = /tmp/mysql.sock

	basedir = /usr/local/mysql
	datadir = /usr/local/mysql/data
	pid-file = /usr/local/mysql/data/mysql.pid
	user = mysql
	bind-address = 0.0.0.0
	server-id = 1

	init-connect = 'SET NAMES utf8mb4'
	character-set-server = utf8mb4

	skip-name-resolve
	#skip-networking
	back_log = 300

	max_connections = 1000
	max_connect_errors = 6000
	open_files_limit = 65535
	table_open_cache = 128 
	max_allowed_packet = 4M
	binlog_cache_size = 1M
	max_heap_table_size = 8M
	tmp_table_size = 16M

	read_buffer_size = 2M
	read_rnd_buffer_size = 8M
	sort_buffer_size = 8M
	join_buffer_size = 8M
	key_buffer_size = 4M

	thread_cache_size = 8

	query_cache_type = 0
	query_cache_size = 0
	query_cache_limit = 2M

	ft_min_word_len = 4

	log_bin = mysql-bin
	binlog_format = mixed
	expire_logs_days = 7 

	log_error = /usr/local/mysql/data/mysql-error.log
	slow_query_log = 1
	long_query_time = 1
	slow_query_log_file = /usr/local/mysql/data/mysql-slow.log

	performance_schema = 0
	explicit_defaults_for_timestamp

	#lower_case_table_names = 1

	skip-external-locking

	default_storage_engine = InnoDB
	#default-storage-engine = MyISAM
	innodb_file_per_table = 1
	innodb_open_files = 500
	innodb_buffer_pool_size = 64M
	innodb_write_io_threads = 4
	innodb_read_io_threads = 4
	innodb_thread_concurrency = 0
	innodb_purge_threads = 1
	innodb_flush_log_at_trx_commit = 2
	innodb_log_buffer_size = 2M
	innodb_log_file_size = 32M
	innodb_log_files_in_group = 3
	innodb_max_dirty_pages_pct = 90
	innodb_lock_wait_timeout = 120

	bulk_insert_buffer_size = 8M
	myisam_sort_buffer_size = 8M
	myisam_max_sort_file_size = 10G
	myisam_repair_threads = 1

	interactive_timeout = 28800
	wait_timeout = 28800

	[mysqldump]
	quick
	max_allowed_packet = 16M

	[myisamchk]
	key_buffer_size = 8M
	sort_buffer_size = 8M
	read_buffer = 4M
	write_buffer = 4M


#以mysql用户帐号的身份建立数据表

	sudo /usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
	#创建管理MySQL数据库的shell脚本
	sudo cp support-files/mysql.server /etc/init.d/mysqld
	sudo chmod +x /etc/init.d/mysqld
	sudo update-rc.d mysqld defaults
	sudo service mysqld start

#MySQL用户设置

	sudo /usr/local/mysql/bin/mysqladmin -u root password 'root'
	sudo /usr/local/mysql/bin/mysql -hlocalhost -u root -proot -P3306 -A -e "grant all privileges on *.* to root@'127.0.0.1' identified by 'root' with grant option;"
	sudo /usr/local/mysql/bin/mysql -hlocalhost -u root -proot -P3306 -A -e "grant all privileges on *.* to root@'localhost' identified by 'root' with grant option;"
	sudo /usr/local/mysql/bin/mysql -hlocalhost -u root -proot -P3306 -A -e "grant all privileges on *.* to root@'%' identified by 'root' with grant option;"
	sudo /us