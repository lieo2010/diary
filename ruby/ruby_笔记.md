## 符号 ##

```ruby
current_situation = :good
puts "Everything is fine" if current_situation == :good
puts "PANIC!" if current_situation == :bad
~#: Everything is fine
上面代码结果等同于
current_situation = good
puts "Everything is fine" if current_situation == good
puts "PANIC!" if current_situation == bad
区别：:good和:bad都是符号。符号不包含值和对象，不像变量那样，而是用作代码中固定的名字，而good和bad再内存中创建了新对象，
而符号仅仅是引用值，只初始化1次。你可能要把符号看成无值的字面常亮，但符号名字是最重要的因素。如果你把:good符号赋值给变
量，并再后面代码中讲该变量与:good相比较，将会得到匹配的结果。因此不需要保存实际值，只需保存概念或选项的情况下，符号很
有用
```
