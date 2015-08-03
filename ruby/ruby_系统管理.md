## 块和yield ##

eg:一个c的迭代代码：
```ruby
int i;
for(i = 0; i< message_count; i++){
		char *m = messages[i];
		puts (m);
}
```
ruby完成上述功能：
```ruby
messages.each{|m| puts m}
```

- ruby块代码编程风格

```ruby
[1,2,3].each do |number|
		puts number
end
或
[1,2,3].each{|number| puts number}

```

