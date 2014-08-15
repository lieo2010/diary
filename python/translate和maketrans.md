# Python——maketrans和translate方法

- - -

## translate ##
S.translate(table [,deletechars]) -> string

Return a copy of the string S, where all characters occurring in the optional argument deletechars are removed, and the remaining characters have been mapped through the given translation table, which must be a string of length 256.

简单来说就是对字符串S移除deletechars包含的字符，然后保留下来的字符按照table里面的字符映射关系映射（比如a变成A，后面会解释到）。那个莫名其妙的"which must be a string of length 256"就不用深究了，反正table就是由string.maketrans方法生成的

## string.maketrans ##

string.maketrans(intab, outtab) --> This method returns a translation table that maps each character in the intab string into the character at the same position in the outtab string. Then this table is passed to the translate() function. Note that both intab and outtab must have the same length.

## 实例说明 ##

```python
import string

s = 'abcdefg-1234567'
table = string.maketrans('', '')  #没有映射，实际上就是按原始字符保留，看下面用到translate中时的效果
s.translate(table) # 输出abcdefg-1234567
s.translate(table, 'abc123') # 输出defg-4567 可以看到删除了字符abc123

#下面是有字符映射的效果

table = string.maketrans('abc', 'ABC')
s.translate(table) # 输出ABCdefg-1234567 就是将abc映射为大写的ABC，前提是abc如果被保留下来了
s.translate(table, 'ab123') # 输出Cdefg-4567 先把s中的ab123去除了，然后在保留下来的字符中应用table中指定的字符映射关系映射：c -> C

解释完毕，现在知道为什么maketrans(intab, outtab)中两个字符串参数长度必须一样了，因为是字符一对一的映射。
```