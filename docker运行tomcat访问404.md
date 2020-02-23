https://hub.docker.com/_/tomcat 里面介绍访问已删除 webapps 目录文件

通过进入容器Tomcat 目录查看,   

有一个空的 WebApps 和 webapps.dist 目录，

将 webapps 目录删除，webapps.dist 目录 重命名为 webapps 目录 重新访问即可访问到 
