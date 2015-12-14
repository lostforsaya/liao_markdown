###记录一次nginx版本发布过程

**操作任务：**  
JJ的nginx新增一个hjy1314的日志轮转配置

1.克隆最新的远程库：
    
    mkdir /tmp/nginx
    cd /tmp/nginx
	git clone git@10.158.2.17:nginx4jj

2.修改nginx的配置：

3.这时候王毅此时在其他地方向远程库提交了新的改动，这时候我拉的副本就不是最新的

	git pull origin master --tags

 这样把远程库最新的改动拉取合并到本地库

4.提交本地库改动到远程库：

	git add filename #如果新增文件，还需要先add到暂存区再提交到远程库 
	git commit -am "add hjy1314 log archive and support by liaoheng Oper_Nginx_JinJing_Main_104"

5.设置上传人邮箱和用户名：

	git config --global user.email "liaoheng@shandagames.com"
	git config --global user.name "liaoheng"

6.为修改后的本地库打上tag:
	
	git tag Oper_Nginx_JinJing_Main_104

7.把标签tag推送到远程库上:
	
	git push --tags origin master

8.deploy.sdo.com -> OP -> nginx4JJ-> NGINX_UPDATE







