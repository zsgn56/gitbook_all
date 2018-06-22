1、python将all文件夹下的所有文本去重

```
# -*- coding: UTF-8 -*-  
import os
def quchong(infile,outfile):

  infopen = open(infile,'r')
  outopen = open(outfile,'w')
  lines = infopen.readlines()
  list_1 = []
  for line in lines:
    if line not in list_1:
      list_1.append(line)
      outopen.write(line)
  infopen.close()
  outopen.close()
#获取目标文件夹的路径
filedir = os.getcwd()+'/all'
filedir_new = os.getcwd()+'/all_new'
#获取当前文件夹中的文件名称列表  
filenames=os.listdir(filedir)
#遍历文件名
for filename in filenames:
    filepath = filedir+'/'+filename
    filepath_new = filedir_new+'/'+filename
    quchong(filepath,filepath_new)

```



