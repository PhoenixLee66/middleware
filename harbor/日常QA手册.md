###### 1. admin 密码重置

```plsql
update registry.harbor_user set salt='', password='' where username='admin';
```


