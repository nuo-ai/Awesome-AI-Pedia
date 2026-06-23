# mac 系统存储满了

du -sh ~/Library/* | sort -hr

rm -rf ~/Library/Caches/*


# 3. 清理日志
rm -rf ~/Library/Logs/*


find ~/Library/Containers -type d -name Caches -exec sh -c 'rm -rf "$1"/*' _ {} \;