###1、reflection+annotation
---
*method.invoke(resource, args);抛出java.lang.IllegalArgumentException*

> annotation

1. Java的泛型是类型擦除的:
Java中的泛型是在编译期间有效的,在运行期间将会被删除,也就是所有泛型参数类型在编译后都会被清除掉.

> 原因：使用了float基本类型


###2、long can note cast to integer
由于mysql的int类型勾选了unsigned，导致int站11位，jbbc转换为long类型：
```
if (!field.isUnsigned() || field.getMysqlType() == MysqlDefs.FIELD_TYPE_INT24) {
    return Integer.valueOf(getInt(columnIndex));
}

return Long.valueOf(getLong(columnIndex));
```
