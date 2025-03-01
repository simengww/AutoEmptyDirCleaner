小白
#!/system/bin/sh

# 日志文件路径
LOG_FILE="/data/adb/modules/AutoEmptyDirCleaner/clean.log"

# 目标目录（可根据需要修改）
TARGET_DIR="/storage/emulated/0"

# 排除目录模式（使用|分隔）
EXCLUDE_PATHS="*/Android|*/Documents|*/Pictures|*/DCIM|*/Download|*/Movies|*/Music|*/Alarms|*/Podcasts|*/Ringtones|*/Notifications"

# 创建日志目录
mkdir -p "$(dirname "$LOG_FILE")"
echo "$(date "+%Y-%m-%d %H:%M:%S") 开始清理..." >> "$LOG_FILE"

# 查找并删除空目录和小文件夹
find "$TARGET_DIR" -type d 2>/dev/null | while read dir; do
  # 跳过排除目录
  if echo "$dir" | grep -qE "$EXCLUDE_PATHS"; then
    continue
  fi
  
  # 检查目录状态
  if [ "$dir" != "$TARGET_DIR" ]; then
    dir_size=$(du -sk "$dir" 2>/dev/null | cut -f1)
    dir_files=$(ls -A "$dir" 2>/dev/null)
    
    # 如果是空目录或者目录大小小于1KB且不包含文件
    if [ -z "$dir_files" ] || ([ "$dir_size" -lt 1 ] && [ -z "$(find "$dir" -type f)"]); then
      rm -rf "$dir"
      echo "$(date "+%Y-%m-%d %H:%M:%S") 已删除：$dir (大小: ${dir_size:-0}KB)" >> "$LOG_FILE"
    fi
  fi
done

echo "$(date "+%Y-%m-%d %H:%M:%S") 清理完成" >> "$LOG_FILE"
