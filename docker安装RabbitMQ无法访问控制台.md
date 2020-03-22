问题：

       rabbitmq 的镜像创建后，镜像启动无法访问控制台。

解决：

       需要进入到容器内部，

       docker exec -it myrabbit /bin/bash

       在 plugins 文件夹下执行命令安装控制台插件：

       rabbitmq-plugins enable rabbitmq_management
