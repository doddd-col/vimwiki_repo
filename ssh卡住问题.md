#本地
#打开
sudo vim /etc/ssh/ssh_config
# 添加
ServerAliveInterval 50		--每隔50秒就向服务器发送一个请求
ServerAliveCountMax 3		--允许超时的次数，一般都会响应 

#远程
# 打开
sudo vim /etc/ssh/sshd_config
# 添加
ClientAliveInterval 50
ClientAliveCountMax 3 
