# jupyterhub-Auth 
## jupyterhub PAM Authenticator  
### 旨在jupyterhub部署后提供基于系统用户验证的多用户jupyter服务，包括在admin页面创建默认密码的新用户用于登录使用
step1:  
vim /opt/miniconda3/lib/python3.7/site-packages/jupyterhub/auth.py 
```
p = Popen(cmd, stdout=PIPE, stderr=STDOUT, shell=True) 
```
step2:  
jupyterhub --generate-config /etc/jupyterhub  
编辑并使用以下配置  
```
c.JupyterHub.ip ='0.0.0.0' #允许远程访问
c.JupyterHub.port = 8888
c.PAMAuthenticator.encoding = 'utf8'
c.LocalAuthenticator.create_system_users = True #允许创建系统用户
c.Authenticator.whitelist = {'jptest1','jptest2','jptest3'} #指定用户白名单
c.Authenticator.admin_users = {'jpadmin'} #需指定管理员用户，后续创建系统用户时使用
c.JupyterHub.statsd_prefix = 'jupyterhub'
c.PAMAuthenticator.open_sessions = False
#useradd -c "jptest5" -g jupyterhub -d /home/jptest5 -s /bin/bash -p "yftian" jptest5 && echo "jptest5:yftian" | chpasswd
c.LocalAuthenticator.add_user_cmd = ["useradd -c USERNAME -g jupyterhub -d /home/USERNAME -s /bin/bash -p yftian USERNAME && echo USERNAME:yftian | chpasswd"] #用于创建系统用户的命令
```
step3:  
启动jupyterhub服务，使用jpadmin在admin界面进行新用户创建  
## Ldap Authenticator  
### 基于openldap中的用户验证，认证通过后会在系统中创建对应的系统用户以及home目录。如果在允许登录的openldap组织中包含该用户，那么该用户直接可以使用其用户名和密码实现jupyterhub登录。
step1:  


