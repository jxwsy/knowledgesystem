# array数组

英文官方文档: [https://docs.python.org/3.8/library/array.html](https://docs.python.org/3.8/library/array.html)

中文官方文档: [https://docs.python.org/zh-cn/3.8/library/array.html](https://docs.python.org/zh-cn/3.8/library/array.html)

```sh
>>> from array import array

# class array.array(typecode[, initializer])
# 一个包含由 typecode 限制类型的条目的新数组，并由可选的 initializer 值进行初始化，该值必须为一个列表、bytes-like object 或包含正确类型元素的可迭代对象。
>>> arr = array('b',[1,2,3,4,5])
>>> print(arr)
array('b', [1, 2, 3, 4, 5])

# 在内部表示中一个数组项的字节长度。
>>> print(arr.itemsize)
1

# array.append(x)
# 添加一个值为 x 的新项到数组末尾。
>>> arr.append(6)
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6])

# array.buffer_info()
# 返回一个元组 (address, length) 以给出用于存放数组内容的缓冲区元素的当前内存地址和长度。 
>>> print(arr.buffer_info())
(2849015194032, 6)

# 数组对象支持普通的序列操作如索引、切片、拼接和重复等。 
# 当使用切片赋值时，所赋的值必须为具有相同类型码的数组对象；
>>> arr[0]
1
>>> arr[0:2]
array('b', [1, 2])
```


```sh
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6])

>>> a = array('b',(1,2,3))

# array.extend(iterable)
# 将来自 iterable 的项添加到数组末尾。 
>>> arr.extend(a)
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6, 1, 2, 3])

>>> arr.extend((1,2,3))
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6, 1, 2, 3, 1, 2, 3])

>>> arr.extend(("a","b"))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: an integer is required (got type str)

# array.count(x)
# 返回 x 在数组中的出现次数。
>>> print(arr.count(1))
3
```


```sh
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6])

# array.index(x)
# 返回最小的 i 使得 i 为 x 在数组中首次出现的序号。
>>> print(arr.index(1))
0
>>> print(arr.index(2))
1

# array.insert(i, x)
# 将值 x 作为新项插入数组的 i 位置之前。 负值将被视为相对于数组末尾的位置。
>>> arr.insert(1,10)
>>> print(arr)
array('b', [1, 10, 2, 3, 4, 5, 6])

# array.pop([i])
# 从数组中移除序号为 i 的项并将其返回。
>>> arr.pop(1)
10
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6])

# array.remove(x)
# 从数组中移除首次出现的 x。
>>> arr.remove(1)
>>> print(arr)
array('b', [2, 3, 4, 5, 6])

# array.reverse()
# 反转数组中各项的顺序。
>>> arr.reverse()
>>> print(arr)
array('b', [6, 5, 4, 3, 2])
```

```sh
>>> arr = array('u',"abcdef")
>>> print(arr)
array('u', 'abcdef')

# array.tobytes()
# 将数组转换为一个机器值数组并返回其字节表示
>>> arrb = arr.tobytes()
>>> print(arrb)
b'a\x00b\x00c\x00d\x00e\x00f\x00'

# array.frombytes(s)
# 添加来自字符串的项，将字符串解读为机器值的数组
>>> arrr = array('u',"g")
>>> arrr.frombytes(arrb)
>>> print(arrr)
array('u', 'gabcdef')
```

```sh
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6])

# array.fromlist(list)
# 添加来自 list 的项。 这等价于 for x in list: a.append(x)，区别在于如果发生类型错误，数组将不会被改变。
>>> arr.fromlist([1,2,3])
>>> print(arr)
array('b', [1, 2, 3, 4, 5, 6, 1, 2, 3])

# array.tolist()
# 将数组转换为包含相同项的普通列表。
>>> arr.tolist()
['a', 'b', 'c', 'd', 'e', 'f']
```

```sh
>>> arrs = array('u',"abcd")
>>> print(arrs)
array('u', 'abcd')

# array.tounicode()
# 将数组转换为一个 Unicode 字符串。 
# 数组必须是类型为 'u' 的数组；否则将引发 ValueError。 
# 请使用 array.tobytes().decode(enc) 来从其他类型的数组生成 Unicode 字符串。
>>> arrs.tounicode()
'abcd'

# array.fromunicode(s)
# 使用来自给定 Unicode 字符串的数组扩展数组。 
# 数组必须是类型为 'u' 的数组；否则将引发 ValueError。 
# 请使用 array.frombytes(unicodestring.encode(enc)) 来将 Unicode 数据添加到其他类型的数组。
>>> arrs.fromunicode("ef")
>>> print(arrs)
array('u', 'abcdef')


```