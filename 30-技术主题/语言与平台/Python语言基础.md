## 格式化输出
```python
name = "张三"
age = 11
score = 99.9

# %操作符
print("个人信息：%s %d %.2f" % (name, age, score))

# format函数
print("个人信息：{} {} {}".format(name, age, score))

# f-strings
print(f"个人信息：{name} {age} {socre}")
```

## 数据类型
python 是弱类型编程语言，变量没有类型，而是指向了一个数据，因此变量可以指向任意数据类型的变量；

```python
# 变量的定义：变量名 = 值
name = "张三"
age = 11
name = 100 # name重新赋值成整数

type(name) # 查看name变量指向的数据的类型
```

| 基本数据类型 | 描述 |
| --- | --- |
| int | 整数 |
| float | 小数 |
| bool | 布尔 |
| string | 字符串 |


| 基本数据类型 | 描述 |
| --- | --- |
| int | 整数 |
| float | 小数 |
| bool | 布尔 |
| string | 字符串 |


