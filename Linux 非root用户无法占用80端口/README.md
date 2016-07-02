# Linux 非root用户无法占用80端口

默认情况下Linux的1024以下端口是只有root用户才有权限占用，所以tomcat，apache，nginx等等
程序如果想要用普通用户来占用80端口的话就会抛出` java.net.BindException: Permission denied`的异常。 

解决办法有三种： 
 
1. 使用非80端口启动程序，然后再用iptables做一个端口转发。 `iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080`用root用户直接去执行就可以了!    
2. 使用root权限直接运行tomcat
3. 使用nginx转发，当然nginx需要使用root权限(推荐)

