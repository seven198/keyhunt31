# =======================
#   KeyHunt 最终稳定版（含快捷命令 + 交互式清理）
# =======================

export DEBIAN_FRONTEND=noninteractive

echo "[1/6] 清理旧环境..."
screen -ls | grep keyhunt | awk '{print $1}' | xargs -r -I{} screen -S {} -X quit
rm -rf /root/keyhunt
rm -f /root/keyhunt_wrapper.sh
rm -f /root/keyhunt_monitor.sh
rm -f /root/keyhunt_live.log
rm -f /root/keyhunt_hit.txt
rm -f /root/msmtp.log

echo "[2/6] 安装依赖..."
apt update -y
apt install -y build-essential libssl-dev libgmp-dev screen msmtp msmtp-mta mailutils

echo "[3/6] 配置 163 发信..."
cat << 'EOF' > /etc/msmtprc
defaults
auth           on
tls            on
tls_starttls   off
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /root/msmtp.log

account        netease
host           smtp.163.com
port           465
from           13537032388@163.com
user           13537032388@163.com
password       VHQjdYyc5d5JGnFa
account default : netease
EOF

chmod 600 /etc/msmtprc
ln -sf /usr/bin/msmtp /usr/sbin/sendmail
ln -sf /usr/bin/msmtp /usr/bin/sendmail

echo "[4/6] 解压 keyhunt 本地仓库..."
cd /root
if [ ! -f "/root/keyhunt_local.tar.gz" ]; then
    echo "[错误] 未找到 keyhunt_local.tar.gz，请上传！"
    exit 1
fi
tar -xzvf keyhunt_local.tar.gz

cd keyhunt
make || make legacy

echo "[5/6] 创建 keyhunt 主脚本..."
cat << 'EOF' > /root/keyhunt_wrapper.sh
#!/bin/bash
BTCADDR="1LhE6sCTuGae42Axu1L1ZB7L96yi9irEBE"
RANGE_FROM="40000000"
RANGE_TO="7fffffff"
THREADS=$(nproc)
STATS=10

cd /root/keyhunt
echo "$BTCADDR" > tests/test1.txt

/root/keyhunt/keyhunt -m address \
    -f tests/test1.txt \
    -r ${RANGE_FROM}:${RANGE_TO} \
    -l compress \
    -t ${THREADS} \
    -R -e -s ${STATS} \
    2>&1 | tee /root/keyhunt_live.log
EOF

chmod +x /root/keyhunt_wrapper.sh

echo "[6/6] 创建监控脚本..."
cat << 'EOF' > /root/keyhunt_monitor.sh
#!/bin/bash
EMAIL="13537032388@163.com"
echo "" > /root/keyhunt_hit.txt

tail -Fn0 /root/keyhunt_live.log | while read line; do
    if echo "$line" | grep -q "Private Key:"; then
        echo "[!] 命中：" > /root/keyhunt_hit.txt
        echo "$line" >> /root/keyhunt_hit.txt
        tail -n 30 /root/keyhunt_live.log >> /root/keyhunt_hit.txt

        mail -s "【KeyHunt 命中提醒】" "$EMAIL" -a "From: 13537032388@163.com" < /root/keyhunt_hit.txt

        pkill keyhunt
        pkill -f keyhunt_wrapper.sh

        exit 0
    fi
done
EOF

chmod +x /root/keyhunt_monitor.sh

echo "[启动后台 keyhunt 与监控...]"
screen -dmS keyhunt /root/keyhunt_wrapper.sh
screen -dmS keymonitor /root/keyhunt_monitor.sh

echo "===================================================="
echo "   KeyHunt 已启动（后台运行）"
echo "===================================================="


# ======================================================
#   安装 7 个快捷命令：key / qt / qzqt / qzjs / zjm / rw / csh
# ======================================================

# key：查询私钥命中
echo '#!/bin/bash' > /usr/local/bin/key
echo 'cat /root/keyhunt_hit.txt' >> /usr/local/bin/key
chmod +x /usr/local/bin/key

# qt：普通唤醒前台
echo '#!/bin/bash' > /usr/local/bin/qt
echo 'screen -r keyhunt' >> /usr/local/bin/qt
chmod +x /usr/local/bin/qt

# qzqt：强制唤醒前台
echo '#!/bin/bash' > /usr/local/bin/qzqt
echo 'screen -dr keyhunt' >> /usr/local/bin/qzqt
chmod +x /usr/local/bin/qzqt

# qzjs：清理所有 screen（全清）
echo '#!/bin/bash' > /usr/local/bin/qzjs
echo 'for S in $(screen -ls | awk "/\./ {print \$1}"); do screen -S "$S" -X quit; done' >> /usr/local/bin/qzjs
chmod +x /usr/local/bin/qzjs

# zjm：后台（等同 Ctrl+A+D）
echo '#!/bin/bash' > /usr/local/bin/zjm
echo 'screen -S keyhunt -X detach' >> /usr/local/bin/zjm
chmod +x /usr/local/bin/zjm

# rw：查询任务数量
echo '#!/bin/bash' > /usr/local/bin/rw
echo 'screen -ls' >> /usr/local/bin/rw
chmod +x /usr/local/bin/rw

# csh：交互式清理日志（Clean Start Helper）
echo '#!/bin/bash' > /usr/local/bin/csh
echo 'FILES=("/root/keyhunt_live.log" "/root/keyhunt_hit.txt" "/root/msmtp.log")' >> /usr/local/bin/csh
echo 'echo "===================================================="' >> /usr/local/bin/csh
echo 'echo "   KeyHunt 初始化（日志清理）"' >> /usr/local/bin/csh
echo 'echo "   将删除以下文件（如果存在）：" ' >> /usr/local/bin/csh
echo 'for f in "${FILES[@]}"; do echo "     - $f"; done' >> /usr/local/bin/csh
echo 'echo "===================================================="' >> /usr/local/bin/csh
echo 'read -p "继续请输入 1 ，取消请输入 2 ：" choice' >> /usr/local/bin/csh
echo 'if [ "$choice" != "1" ]; then echo "已取消，不做任何修改。"; exit 0; fi' >> /usr/local/bin/csh
echo 'echo "开始删除..."' >> /usr/local/bin/csh
echo 'for f in "${FILES[@]}"; do' >> /usr/local/bin/csh
echo '    if [ -f "$f" ]; then rm -f "$f"; echo "已删除：$f"; else echo "不存在：$f"; fi' >> /usr/local/bin/csh
echo 'done' >> /usr/local/bin/csh
echo 'echo "初始化完成。"' >> /usr/local/bin/csh
echo 'echo "===================================================="' >> /usr/local/bin/csh
chmod +x /usr/local/bin/csh


# ======================================================
#   删除 z 的快捷键绑定（保证不再生效）
# ======================================================
sed -i '/bind z detach/d' /root/.screenrc

echo "===================================================="
echo "脚本安装完成！"
echo "快捷命令：key / qt / qzqt / qzjs / zjm / rw / csh"
echo "===================================================="
