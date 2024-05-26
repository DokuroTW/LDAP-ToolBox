# LDAP-ToolBox
LDAP的工具箱

在這邊保留了LDAP常使用的command與範本

# 確認olcModuleLoad
slapcat -n 0 -F /etc/ldap/slapd.d/ | grep olcModuleLoad

# ldapsearch

查詢config中所有dn
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn
查询整个目录
ldapsearch -x -H ldapi:/// -b "dc=example,dc=com"
查询特定用户
ldapsearch -x -H ldapi:/// -b "dc=example,dc=com" "(uid=john.doe)"
使用DN和密码进行认证查询
ldapsearch -x -D "cn=admin,dc=example,dc=com" -w admin_password -b "dc=example,dc=com"

# ldapadd
添加一个新用户
ldapadd -x -D "cn=admin,dc=example,dc=com" -w admin_password -f newuser.ldif

# ldapmodify
修改用户的属性
ldapmodify -x -D "cn=admin,dc=example,dc=com" -w admin_password -f modifyuser.ldif

# ldappasswd
设置特定用户的密码
ldappasswd -x -D "cn=admin,dc=example,dc=com" -w admin_password -S -s new_password "uid=john.doe,ou=people,dc=example,dc=com"

# ldapdelete
删除特定用户
ldapdelete -x -D "cn=admin,dc=example,dc=com" -w admin_password "uid=john.doe,ou=people,dc=example,dc=com"

#-------------------------------------------------------------------------------------------------------------------#
# -Y EXTERNAL

查看LDAP服务器配置
ldapsearch -Y EXTERNAL -H ldapi:/// -b "cn=config"

修改LDAP配置
ldapmodify -Y EXTERNAL -H ldapi:/// -f change.ldif

添加新的Schema或Overlay
ldapadd -Y EXTERNAL -H ldapi:/// -f new-schema.ldif

#-------------------------------------------------------------------------------------------------------------------#
# 密碼事件處裡

分析錯誤
ldapwhoami -x -D "cn=,ou=,dc=,dc=" -W -e ppolicy -v
ldapsearch -x -ALL udi= +

鎖定解除
cat << EOF | ldapadd -x -H ldapi:/// -D"cn=,dc=,dc=" -W
dn: udi=,ou=,dc=,dc=
changetype: modify
delete: pwdAccountLockedTime
EOF

再次取得帳號
ldapwhoami -x -D "uid=,ou=,dc=,dc=" -W -e ppolicy -v

提示修該初始密碼
cat << EOF | ldapadd -x -H ldapi:/// -D "cn=,dc=,dc=" -W 
dn: uid=,ou=,dc=,dc=
changetype: modify
replace: pwdReset
EOF

密碼過期
cat << EOF | ldapmodify -x -H ldapi:/// -D cn=config -W
dn: uid=,ou=,dc=,dc=
changetype: modify
delete: userpassword
userpassword: 12345
-
add: userpassword
userpassword: 12345
