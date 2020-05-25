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
pip install jupyterhub-ldapauthenticator  
step2:  
修改anaconda_home/lib/python3.6/site-packages/ldapauthenticator目录下的ldapauthenticator.py文件  
1)文件头添加代码 import pwd,os  
2)添加两个方法:  
```
    def system_user_exists(self, username):
        try:
            pwd.getpwnam(username)
        except KeyError:
            return False
        else:
            return True

    def add_system_user(self, username, password):
        res = os.system('useradd  %(name1)s -s /bin/nologin' % {'name1': username})
        if res:
            self.log.warn('user %s create failure' % username)
            return False
        res = os.system('echo %(pass)s |passwd --stdin %(name1)s' % {'name1': username, 'pass': password})
        if res:
            self.log.warn('user %s password create failure' % username)
            return False
        return True
```
3)authenticate函数里面的 return username 前面添加以下代码:
```
        if not self.system_user_exists(username):
            res = self.add_system_user(username, password)
            if not res:
                return None
```
[详见ldapauthenticator.py]()
step3:  
编辑并使用以下配置  
```
c.JupyterHub.ip ='0.0.0.0'
c.JupyterHub.port = 8888
c.Authenticator.admin_users = ['c1701']
c.LocalAuthenticator.create_system_users = True
c.JupyterHub.authenticator_class = 'ldapauthenticator.LDAPAuthenticator'
c.LDAPAuthenticator.server_address = '192.168.228.130'  #ldap服务器地址
c.LDAPAuthenticator.server_port = 389                #ldap端口
c.LDAPAuthenticator.bind_dn_template = [             #template以实际为准
        "uid={username},ou=test,dc=xxx,dc=xxx",
        "uid={username},ou=Software_Engine,dc=xxx,dc=xxx"
]
```  
这样在ldap认证通过以后如果本地没有这个用户，就会自动创建。但是这种做法在ldap删除用户以后无法自动删除本地用户，这也是jupyterhub官方不支持创建用户的原因。
