<h1>Recommended MySQL System Variable Settings for SSD</h1>
<tbody><tr class="tableHeader tableFixed"><th colspan="2">MySQL Tuning (/etc/my.cnf) - [ <a class="linkOverDark" href="https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html">Documentation</a> ] Note: Many changes below require a database restart</th></tr><tr class="even tableRow">
<td colspan="2" style="text-align:left;padding:0px"><table id="mysql" class="Table" style="width:100%"><thead><tr class="tableHeader">  <th class="tableSubHeaderColumn">Variable</th>  <th class="tableSubHeaderColumn right">Default Value</th>  <th class="tableSubHeaderColumn center">&nbsp;</th>  <th class="tableSubHeaderColumn">Recommended Value</th>  <th class="tableSubHeaderColumn">Comments</th></tr></thead><tbody><tr class="odd tableRow">
<td>version</td><td class="right ">MySQL 5.6+</td><td class="center">&gt;=</td><td>5.6</td><td class="">MySQL 5.6+ and MariaDB 10.0+ are great releases, and are very good versions to choose. Make sure you run the very latest release though which fixes a long standing low level networking issue that was causing spine many issues with reliability.</td></tr>
<tr class="even tableRow">
<td>collation_server</td><td class="right textError">utf8mb4_0900_ai_ci</td><td class="center">=</td><td>utf8mb4_unicode_ci</td><td class="textError">When using Apps with languages other than English, it is important to use the utf8mb4_unicode_ci collation type as some characters take more than a single byte.</td></tr>
<tr class="odd tableRow">
<td>character_set_client</td><td class="right ">utf8mb4</td><td class="center">=</td><td>utf8mb4</td><td class="">When using Apps with languages other than English, it is important to use the utf8mb4 character set as some characters take more than a single byte.</td></tr>
<tr class="even tableRow">
<td>max_connections</td><td class="right ">151</td><td class="center">&gt;=</td><td>100</td><td class="">Depending on the number of logins and use of spine data collector, MySQL will need many connections.  The calculation for spine is: total_connections = total_processes * (total_threads + script_servers + 1), then you must leave headroom for user connections, which will change depending on the number of concurrent login accounts.</td></tr>
<tr class="odd tableRow">
<td>max_allowed_packet</td><td class="right ">67108864</td><td class="center">&gt;=</td><td>16777216</td><td class="">With Remote polling capabilities, large amounts of data will be synced from the main server to the remote pollers.  Therefore, keep this value at or above 16M.</td></tr>
<tr class="even tableRow">
<td>max_heap_table_size</td><td class="right ">16M</td><td class="center">&gt;=</td><td>15M</td><td class="">If using the Apps Performance Booster and choosing a memory storage engine, you have to be careful to flush your Performance Booster buffer before the system runs out of memory table space.  This is done two ways, first reducing the size of your output column to just the right size.  This column is in the tables poller_output, and poller_output_boost.  The second thing you can do is allocate more memory to memory tables.  We have arbitrarily chosen a recommended value of 10% of system memory, but if you are using SSD disk drives, or have a smaller system, you may ignore this recommendation or choose a different storage engine.  You may see the expected consumption of the Performance Booster tables under Console -&gt; System Utilities -&gt; View Boost Status.</td></tr>
<tr class="odd tableRow">
<td>tmp_table_size</td><td class="right ">16M</td><td class="center">&gt;=</td><td>15M</td><td class="">When executing subqueries, having a larger temporary table size, keep those temporary tables in memory.</td></tr>
<tr class="even tableRow">
<td>join_buffer_size</td><td class="right textWarning">0M</td><td class="center">&gt;=</td><td>31M</td><td class="textWarning">When performing joins, if they are below this size, they will be kept in memory and never written to a temporary file.</td></tr>
<tr class="odd tableRow">
<td>innodb_file_per_table</td><td class="right ">ON</td><td class="center">=</td><td>ON</td><td class="">When using InnoDB storage it is important to keep your table spaces separate.  This makes managing the tables simpler for long time users of MySQL.  If you are running with this currently off, you can migrate to the per file storage by enabling the feature, and then running an alter statement on all InnoDB tables.</td></tr>
<tr class="even tableRow">
<td>innodb_buffer_pool_size</td><td class="right textWarning">128M</td><td class="center">&gt;=</td><td>238M</td><td class="textWarning">InnoDB will hold as much tables and indexes in system memory as is possible.  Therefore, you should make the innodb_buffer_pool large enough to hold as much of the tables and index in memory.  Checking the size of the /var/lib/mysql/database_name directory will help in determining this value.  We are recommending 25% of your systems total memory, but your requirements will vary depending on your systems size.</td></tr>
<tr class="odd tableRow">
<td>innodb_doublewrite</td><td class="right ">ON</td><td class="center">=</td><td>ON</td><td class="">This settings should remain ON unless your Apps instances is running on either ZFS or FusionI/O which both have internal journaling to accomodate abrupt system crashes.  However, if you have very good power, and your systems rarely go down and you have backups, turning this setting to OFF can net you almost a 50% increase in database performance.</td></tr>
<tr class="even tableRow">
<td>innodb_lock_wait_timeout</td><td class="right ">50</td><td class="center">&gt;=</td><td>50</td><td class="">Rogue queries should not for the database to go offline to others.  Kill these queries before they kill your system.</td></tr>
<tr class="odd tableRow">
<td>innodb_flush_method</td><td class="right ">fsync</td><td class="center">eq</td><td>O_DIRECT</td><td class="">Maximum I/O performance happens when you use the O_DIRECT method to flush pages.</td></tr>
<tr class="even tableRow">
<td>innodb_flush_log_at_timeout</td><td class="right textError">1</td><td class="center">&gt;=</td><td>3</td><td class="textError">As of MySQL 8.0.29-0ubuntu0.20.04.3, the you can control how often MySQL flushes transactions to disk.  The default is 1 second, but in high I/O systems setting to a value greater than 1 can allow disk I/O to be more sequential</td></tr>
<tr class="odd tableRow">
<td>innodb_read_io_threads</td><td class="right textError">4</td><td class="center">&gt;=</td><td>32</td><td class="textError">With modern SSD type storage, having multiple read io threads is advantageous for applications with high io characteristics.</td></tr>
<tr class="even tableRow">
<td>innodb_write_io_threads</td><td class="right textError">4</td><td class="center">&gt;=</td><td>16</td><td class="textError">With modern SSD type storage, having multiple write io threads is advantageous for applications with high io characteristics.</td></tr>
<tr class="odd tableRow">
<td>innodb_buffer_pool_instances</td><td class="right textWarning">1</td><td class="center">&gt;=</td><td>2</td><td class="textWarning">MySQL will divide the innodb_buffer_pool into memory regions to improve performance.  The max value is 64.  When your innodb_buffer_pool is less than 1GB, you should use the pool size divided by 128MB.  Continue to use this equation upto the max of 64.</td></tr>
<tr class="even tableRow">
<td>innodb_io_capacity</td><td class="right textWarning">200</td><td class="center">&gt;=</td><td>5000</td><td class="textWarning">If you have SSD disks, use this suggestion.  If you have physical hard drives, use 200 * the number of active drives in the array.  If using NVMe or PCIe Flash, much larger numbers as high as 100000 can be used.</td></tr>
<tr class="odd tableRow">
<td>innodb_io_capacity_max</td><td class="right textWarning">2000</td><td class="center">&gt;=</td><td>10000</td><td class="textWarning">If you have SSD disks, use this suggestion.  If you have physical hard drives, use 2000 * the number of active drives in the array.  If using NVMe or PCIe Flash, much larger numbers as high as 200000 can be used.</td></tr>
</tbody></table></td></tr>
</tbody></table></div>
