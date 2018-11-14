---
layout: post
title: "Got an error creating the test database: permission denied to create database"
description: "postgresql error"
date: 2018-11-08
tags: [error, tip]
comments: true
share: true
---



테스트 데이터베이스 생성하다 에러 발생

```bash
Got an error recreating the test database: must be owner of database <db_name>
```

해결방법: 테스트 db의 owner에게 create db권한을 줌

```sq;
ALTER USER <user_name> CREATEDB;
```



또 다른 에러

```bash
Got an error recreating the test database: must be owner of database <db_name>
```

해결 방법:  owner 권한을 줌

```sql
ALTER DATABASE <db_name> OWNER TO <user_name>;
```

