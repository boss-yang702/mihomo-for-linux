curl “订阅地址url -o conf/config.yaml

# 启动
```sh
nohup ./mihomo -d conf > mihomo.log 2>&1 &
```

# 设置代理
--需要设置自己相应的地址和端口--

```sh
export HTTP_PROXY="http://127.0.0.1:7897" 
export HTTPS_PROXY="http://127.0.0.1:7897"
```

python代码中
```python
import os
os.environ["HTTP_PROXY"] = "http://127.0.0.1:7897"
os.environ["HTTPS_PROXY"] = "http://127.0.0.1:7897"
```
# 检测是否启动成功
```sh
python -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='bert-base-uncased', filename='config.json')"
```
# Linux 系统 开机自启动
```sh
sudo vim /etc/init.d/mihomo
```
写入以下内容（按需修改 ExecStart 路径和参数）：
```sh
#!/bin/bash
### BEGIN INIT INFO
# Provides:          mihomo
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop:     $network $local_fs $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Mihomo Proxy Service
# Description:       Run Mihomo as a daemon
### END INIT INFO

NAME="mihomo"
PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin"
APP="/path/to/mihomo"          # 替换为 mihomo 可执行文件路径
CONF_DIR="/path/to/conf"       # 替换为配置文件目录
LOG_FILE="/var/log/mihomo.log"
PID_FILE="/var/run/mihomo.pid"

case "$1" in
    start)
        echo "Starting $NAME..."
        nohup $APP -d $CONF_DIR > $LOG_FILE 2>&1 &
        echo $! > $PID_FILE
        ;;
    stop)
        echo "Stopping $NAME..."
        kill -9 $(cat $PID_FILE) 2>/dev/null
        rm -f $PID_FILE
        ;;
    restart)
        $0 stop
        sleep 1
        $0 start
        ;;
    status)
        if [ -f $PID_FILE ] && ps -p $(cat $PID_FILE) > /dev/null; then
            echo "$NAME is running (PID: $(cat $PID_FILE))"
        else
            echo "$NAME is not running"
        fi
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
exit 0
```
```
sudo chmod +x /etc/init.d/mihomo
sudo update-rc.d mihomo defaults  # Debian/Ubuntu
sudo chkconfig --add mihomo       # CentOS 6/RedHat

命令	作用
sudo service mihomo start	启动服务
sudo service mihomo stop	停止服务
sudo service mihomo restart	重启服务
sudo service mihomo status	查看状态
```