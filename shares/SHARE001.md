# MySQL通过frm、ibd文件恢复innodb数据

> 总有需要恢复数据的情况，本文讨论通过frm、ibd文件恢复innodb数据的思路与方法
>
> 使用此方法需要满足以下两个条件：
>
> 1. MySQL使用innodb引擎进行存储，使用myisam引擎存储的只需将文件直接复制到指定目录即可恢复。
> 2. MySQL中必须按表存放数据。若不是，则该方法不适用。

## 一、找回表结构

1. 创建空数据库。
2. 创建表结构，表名与欲恢复的表名一致。表里的字段无所谓。实践过程中发现5.7版本(以下实践均在5.7版本上)，需要主键、索引、字段数量需与原表一致，否则`show create table XXX;`时执行不成功。此时一般情况下我们先创建有一个主键的表，这样成功概率比较大。
3. 关闭MySQL。
4. 将需要恢复的frm文件覆盖刚新建的frm文件。
5. 修改/etc/my.cnf里innodb_force_recovery=1， 如果不成修改为 2,3,4,5,6。
6. 启动MySQL，此时会发现大部分表结构并不能查询出来，通过查找日志发关键信息`InnoDB:\s*Table(?P<db>.*)/(?P<tbname>[^\s]*).*but\s*(?P<num>\d+)\s*columns`，提示字段数量不一致。这步根据字段数量调整步骤2的建表语句，补足字段重复2-5过程，直到`show create table XXX;`能正确显示建表语句。保存建表语句备用。

## 二、找回数据

1. 创建空数据库，导入上一步骤获得表结构。

2. 将当前表空间废弃掉，使当前.ibd数据文件与.frm表结构文件分离。

   `ALTER TABLE ax_table DISCARD TABLESPACE;`

3. 将需要恢复的.ibd数据文件复制至新的表结构文件夹下。导入表空间，使.ibd数据文件与.frm表结构文件关联。

   `ALTER TABLE ax_table IMPORT TABLESPACE;* `

4. 修改/etc/my.cnf文件的 `innodb_force_recovery=1`(如不起效可试试 2,3,4,5,6)，直到可以查询出数据为止。

5. 将该数据库dump出来，至此备份完毕，可以正常新建空数据库然后进行导入。

> tips:以上两个步骤切记不可在生产环境上使用，以免造成不必要的损失。需要独立的环境进行恢复操作，最终得到dump出的备份文件供生产环境恢复使用。  



## 三、编程思路

可以将以上步骤使用shell或者Python等写成恢复脚本进行自动化操作。

在此仅提供思路，其中比较关键复杂的获得表结构的部分代码列出来，完整脚本待完成之后适时放上来。

```python
def parses_tables(dbconf, real=False):
    tables = {}
    for fn in os.listdir(dbconf['dbname']):
        if fn.endswith(".ibd"):
            table = {"dbname": dbconf['dbname'],
                     "tbname": fn.replace('.ibd', '')}
            tables[table['tbname']] = table
    with open(dbconf['log'], 'r') as f:
        for line in f.readlines():
            result = re.search(
                'InnoDB:\s*Table(?P<db>.*)/(?P<tbname>[^\s]*).*but\s*(?P<num>\d+)\s*columns', line)
            if result:
                table = result.groupdict()
                table['num'] = int(table['num'])
                if tables[table['tbname']]:
                    tables[table['tbname']].update(table)
    for tbname, table in six.iteritems(tables):
        columns = ["`id` int(8) unsigned NOT NULL AUTO_INCREMENT"]
        if table.get('num'):
            for i in range(1, int(table['num'])):
                columns.append("`column{i}` int(8)".format(i=i))
        columns.append("PRIMARY KEY(`id`)")
        table['columns'] = ",".join(columns)

    if real:
        conn = myconn(dbconf)
        try:
            with conn.cursor() as cursor:
                for tbname, table in six.iteritems(tables):
                    cursor.execute(SQL_SHOW_CREATE_TABLE.format(**table))
                    tbstruct = cursor.fetchone()
                    if tbstruct:
                        table['sql'] = tbstruct['Create Table']
        finally:
            conn.close()

    return tables
```

> 以上基于网络上搜索到的文章并基于自己实际总结而出。不足之处欢迎指正。