# Common Error

这里是我的typo，username,

solution: 检查模型字段名称，我这里是字段名称输入错误。

```python
>>> use = User.create(password='123456', updated=datetime.datetime.now(), permission=0, is_local=True)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    use = User.create(password='123456', updated=datetime.datetime.now(), permission=0, is_local=True)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 5454, in create
    inst.save(force_insert=True)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 5577, in save
    pk_from_cursor = self.insert(**field_dict).execute()
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 1574, in inner
    return method(self, database, *args, **kwargs)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 1645, in execute
    return self._execute(database)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2288, in _execute
    return super(Insert, self)._execute(database)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2063, in _execute
    cursor = database.execute(self)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2653, in execute
    return self.execute_sql(sql, params, commit=commit)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2647, in execute_sql
    self.commit()
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2438, in __exit__
    reraise(new_type, new_type(*exc_args), traceback)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 177, in reraise
    raise value.with_traceback(tb)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/peewee.py", line 2640, in execute_sql
    cursor.execute(sql, params or ())
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/cursors.py", line 170, in execute
    result = self._query(query)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/cursors.py", line 328, in _query
    conn.query(q)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/connections.py", line 516, in query
    self._affected_rows = self._read_query_result(unbuffered=unbuffered)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/connections.py", line 727, in _read_query_result
    result.read()
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/connections.py", line 1066, in read
    first_packet = self.connection._read_packet()
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/connections.py", line 683, in _read_packet
    packet.check_error()
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/protocol.py", line 220, in check_error
    err.raise_mysql_exception(self._data)
  File "/home/ml/.local/share/virtualenvs/dp_ops-qXNnJ9Is/lib/python3.6/site-packages/pymysql/err.py", line 109, in raise_mysql_exception
    raise errorclass(errno, errval)
peewee.InternalError: (1364, "Field 'username' doesn't have a default value")
```