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
## Ruby程序区分运行来源##

当我们在写模块的时候，或多或少需要直接运行这个文件也可以执行一些方法，但是这样对于当这个模块被require或者include时，显得
不好，在ruby里，有没有区分运行来自当前文件，还是被require的目标文件调用呢？

Python可以
```python
if __name__ == '__main__':
		print "from direct running"
```

Ruby当然也可以
其原理就是判断启动文件是否为模块的代码文件。
```ruby
if __FILE__ ==$0
		puts 'called from direct running'
end
```
举例：工具类模块utils.rb

```ruby
module Utils
    class StringUtils
        def self.test
            puts "test method myfile=" + __FILE__ + ';load from ' +  $0
        end
    end
end

if __FILE__ == $0
    puts 'called from direct running'
    Utils::StringUtils.test()
end
```
直接运行，结果,if条件成立，执行了输出
```
20:04:37-androidyue~/rubydir/test$ ruby utils.rb
called from direct running
test method myfile=utils.rb;load from utils.rb
```

引用Utils的类test.rb
```ruby
require './utils'
Utils::StringUtils.test()
```
运行结果，引入模块的条件不成立，没有输出called from direct running
```
20:08:07-androidyue~/rubydir/test$ ruby test.rb
test method myfile=/home/androidyue/rubydir/test/utils.rb;load from test.rb
```
