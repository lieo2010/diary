# 关闭Mac上的虚拟内存
---
## 查看正在使用的虚拟内存 ##
```ruby
sysctl vm.swapusage
vm.swapusage: total = 1024.00M  used = 354.50M  free = 669.50M  (encrypted)
```

## 关闭虚拟内存后，系统还保留着这些作虚拟内存的交换文件，可以删除
```ruby
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist
sudo rm /private/var/vm/swapfile*
```

## 如果遇到系统不稳定，可以重新开启
```ruby
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist
```
