 1、拷贝旧项目为新项目

 cd /data/gitlab/repositories/simba   \#进入要拷贝的新group

 cp -p -rf backend-sfs.git backend-clean.git

  gitlab-rake gitlab:import:repos

