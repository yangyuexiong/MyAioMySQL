# MyAioMySQL

基于aiomysql封装简化在async.run与aio web框架中的使用

### 直接连接执行sql查询

- 详细注释 [db.py](main/db.py) 中 `MyAioMySQL.query`或`MyAioMySQL.__query`
- 单元测试 [test.py](test/test.py)

```python3
MYSQL_CONF = {
    "user": "root",
    "password": "12345678",
    "host": "127.0.0.1",
    "port": 3306,
    "db": "test",
    "autocommit": "true"
}

db = MyAioMySQL(conf_dict=MYSQL_CONF, debug=True)
sql = "SELECT id, username FROM `user` limit 0,6;"

# 返回唯一对象
result1 = asyncio.run(db.query(sql, only=True))
print(result1, type(result1))

# 返回size长度对象`list` 如 [{"username":"123"},{"username":"123"},{"username":"123"}]
result2 = asyncio.run(db.query(sql, size=3))
print(result2, type(result2), len(result2))

# 根据sql语句实际查询结果返回
result3 = asyncio.run(db.query(sql))
print(result3, type(result3), len(result3))
```

### 直接连接执行sql查询以外的语句例如 update, instr, delete ...

- 详细注释 [db.py](main/db.py) 中 `MyAioMySQL.execute`或`MyAioMySQL.__execute`
- 单元测试 [test.py](test/test.py)

```python
db = MyAioMySQL(conf_dict=MYSQL_CONF, debug=True)
update_sql = """UPDATE `user` SET username = 'aaa' WHERE id = '1';"""
update_result = asyncio.run(db.execute(sql=update_sql))
print(update_result)
```

### 连接池执行sql查询

- 详细注释 [db.py](main/db.py) 中 `MyAioMySQL.pool_query`(查询结果处理与`query`一致)
- 单元测试 [test.py](test/test.py)

```python
MYSQL_CONF = {
    "user": "root",
    "password": "12345678",
    "host": "127.0.0.1",
    "port": 3306,
    "db": "test",
    "autocommit": "true"
}
db = MyAioMySQL(conf_dict=MYSQL_CONF, debug=True)
sql = "SELECT id, username FROM `user` limit 0,6;"
result1 = asyncio.run(db.pool_query(sql, only=True))
print(result1, type(result1))

result2 = asyncio.run(db.pool_query(sql, size=3))
print(result2, type(result2), len(result2))

result3 = asyncio.run(db.pool_query(sql))
print(result3, type(result3), len(result3))
```

### 连接池执行sql查询以外的语句例如 update, instr, delete ...

- 详细注释 [db.py](main/db.py) 中 `MyAioMySQL.pool_execute`(执行结果处理与`execute`一致)
- 单元测试 [test.py](test/test.py)

```python
db = MyAioMySQL(conf_dict=MYSQL_CONF, debug=True)
update_sql = """UPDATE `user` SET username = 'aaa' WHERE id = '1';"""
update_result = asyncio.run(db.pool_execute(sql=update_sql))
print(update_result)
```

### 在aio web框架中使用

- 详细注释 [db.py](main/db.py) 中 `MyAioMySQL.init_pool`
- 详细实现(全局注册)：
    - https://github.com/yangyuexiong/AioHttp_BestPractices/blob/main/registry/db_register.py
    - https://github.com/yangyuexiong/AioHttp_BestPractices/blob/main/ApplicationExample.py

- 详细实现(调用)：
    - https://github.com/yangyuexiong/AioHttp_BestPractices/blob/main/app/api/login/login.py

```python
# 全局注册
async def register_db(app):
    aio_mysql_conf = {
        "host": "...",
        "password": "...",
        ...
    }
    app['aio_mysql_engine'] = await aiomysql.create_pool(**aio_mysql_conf, charset='utf8', loop=app.loop)
    yield
    app['aio_mysql_engine'].close()
    await app['aio_mysql_engine'].wait_closed()


async def create_app():
    app = web.Application()
    ...
    ...
    app.cleanup_ctx.append(register_db)  # aio mysql pool 注册
    return app


# 调用
pool = self.request.app['aio_mysql_engine']
db = MyAioMySQL(pool=pool)
db.pool_query(sql='select...')
db.pool_execute(sql='update...')
```

### 备注

- 调试完毕后，可以将`debug`至空避免信息打印