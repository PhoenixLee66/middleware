###### 1. admin 密码重置

1. 停服务，同时因为权限问题，需要root用户登录

```shell
docker stop java_nexus3

docker exec -u root -it java_nexus3 /bin/bash
```

2. 进入OrientDB控制台

```shell
cd /opt/sonatype/

/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-2.el8_3.x86_64/jre/bin/java -jar nexus/lib/support/nexus-orient-console.jar 
```

3. 进入数据库

```shell
# 相对路径
connect plocal:sonatype-work/nexus3/db/security admin admin
```

4. 重置密码

```sql
update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" WHERE id="admin";
commit;
```

5. 修改文件属性

```shell
# 根据实际情况调整用户名、用户组，以及宿主机目录
chown 200:200 /live/nexus3/data/db/security/*
```

6. 重启nexus3服务
