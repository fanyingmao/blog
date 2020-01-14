---
title: mongodb传参数生成sql语句实现
date: 2018-4-19 11:01:22
tags:  
- mongodb
- sql
---
## 背景
项目由node+mongodb开发改为，java+mysql，为了方便迁移，想将mongodb传参数生成mysql对应的sql语句。
## 实现
具体看[github](https://github.com/fanyingmao/ym-mongodb-sql)
```js
    strChang(obj) {
        if (typeof obj === 'string') {
            return "'" + obj + "'"
        }
        else {
            return obj;
        }
    };

  /**
   * 生成sql查询语句
   * @param tableName 表名
   * @param query 相等查询
   * @param fields 字段筛选
   * @param optSql 其它sql语句
   * @returns {*} promise结果返回
   */
  find(tableName, query, fields, optSql) {
    let fieldsKeys;
    if (!fields) {
      fieldsKeys = [];
    }
    else {
      fieldsKeys = Object.keys(fields);
    }
    let fieldSql;
    if (fieldsKeys.length === 0) {
      fieldSql = '*';
    }
    else {
      fieldSql = fieldsKeys.join(',');
    }

    let queryKeys = Object.keys(query);
    let querySql;

    if (queryKeys.length === 0) {
      querySql = '';
    }
    else {
      querySql = ' where ';
      let queryArr = [];
      queryKeys.forEach(key => {
        let value = query[key];
        switch (typeof value) {
          case 'object':
            let valueKeys = Object.keys(value);
            valueKeys.forEach(key2 => {
              switch (key2) {
                case '$in'://只对in处理其它的以此类推
                  let arr = value[key2];
                  arr.forEach(index => {
                    if (typeof arr[index] === 'string') {
                      arr[index] = this.strChang(arr[index]);
                    }
                  });
                  queryArr.push(key + ' in (' + arr.join(',') + ')');
                  break;
              }
            });
            break;
          default:
            value = this.strChang(value);
            queryArr.push(key + ' = ' + value);
            break;
        }
      });
      querySql += queryArr.join(' and ');
    }
    if (!optSql) {
      optSql = '';
    }
    else {
      optSql = ' ' + optSql;
    }
    let sql = 'select ' + fieldSql + ' from ' + tableName + ' ' + querySql + optSql + ';';
    console.log('sql : ' + sql);
    return this.query(sql, []);
  };
```