# CentOS 7安装gitlab-ee企业版 {#centos-7安装gitlab-ee企业版}

官方：[https://about.gitlab.com/install/\#centos-7](https://about.gitlab.com/install/#centos-7)

# git 项目迁移到 gitlab-ee {#centos-7安装gitlab-ee企业版}

参考：

[https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/import.md](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/import.md)

[https://gitlab.com/iv-mexx/gitlab-migrator](https://gitlab.com/iv-mexx/gitlab-migrator)

```
scp -r quark 192.168.100.10:/var/opt/gitlab/git-data/repositories/
chown -R git:git /var/opt/gitlab/git-data/repositories/
gitlab-rake gitlab:import:repos['/var/opt/gitlab/git-data/repositories']
gitlab-ctl reconfigure

```



