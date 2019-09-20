# python通过配置文件连接mysql

之前在 `PyQt` 中连接 `mysql` 是直接在 `python` 文件中写的，这次把数据库信息放在单独一个配置文件中，省略部分代码：

```python
import configparser
# 生成config对象
conf = configparser.ConfigParser()
# 用config对象读取配置文件
conf.read("test.cfg")
# 以列表形式返回所有的section
sections = conf.sections()
print("sections:", sections)  # sections: ['sec_b', 'Option']
# 得到指定section的所有option
options = conf.options("Option")
print("options:", options)  # options: ['a_key1', 'a_key2']
# 得到指定section的所有键值对
kvs = conf.items("Option")
print("Option:", kvs)  # Option: [('a_key1', '20'), ('a_key2', '10')]
# 指定section，option读取值
host = conf.getint("Option", "host")

pymysql.connect(host,
                                 port=3306,
                                 user=dbStaticResult['user'],
                                 password=dbStaticResult['password'],
                                 db=dbStaticResult['database'],
                                 charset='utf8',
                                 cursorclass=pymysql.cursors.DictCursor)
```
但是这里 `host` 会报 `Servname not supported for ai_socktype` , 读取数据类型不一致导致，所以我们这里用 `json` 来保存数据库信息：
```python
import MySQLdb
import json

cur_time="\""+time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time()))+"\""
print cur_time

task_id = raw_input("input task_id:")

with open('conf/default.conf','r') as confFile:
    confStr = confFile.read()
conf = json.JSONDecoder().decode(confStr);

##########################################
#connect table result
dbStaticResult = conf['database']['db_bim_rap_result_db'];
conn = MySQLdb.connect(host=dbStaticResult['host'],\
        user=dbStaticResult['user'],\
        passwd=dbStaticResult['password'],\
        db=dbStaticResult['database'],\
        port=dbStaticResult['port']);
cur = conn.cursor();
```

配置文件内容如下：
```json
{
  "database":{
    "task":{
        "host":"st01-nsbk-201106-zf-28.st01.****.com",
        "port":3306,
        "database":"bim_statistic_strategy_db",
        "user":"*****",
        "password":"******"
    }
  }
}
```
