###10、异常
---
java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x9C' for column 'content' at row 1.

产生这种异常的原因在于，mysql中的utf8编码最多会用3个字节存储一个字符，如果一个字符的utf8
编码占用4个字节（最常见的就是ios中的emoji表情字符），那么在写入数据库时就会报错。

mysql从5.5.3版本开始，才支持4字节的utf8编码，编码名称为utf8mb4（mb4的意思是max bytes 4），这种编码方式最多用4个字节存储一个字符。

[mysql中Incorrect string value乱码问题解决方案](https://my.oschina.net/lixin91/blog/639270)