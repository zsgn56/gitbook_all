1、python将all文件夹下的所有文本去重，输出到all\_new

```py
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

2、对于文本中第一列存在重复的，只输出第一个

```py

# -*- coding: UTF-8 -*-  
import os
def quchong(infile,outfile):

  infopen = open(infile,'r')
  outopen = open(outfile,'w')
  lines = infopen.readlines()
  list_2 = []
  for line in lines:
    line2 = line.split()
    if line2[0]+'\n' not in list_2:
      list_2.append(line2[0]+'\n')
      outopen.write(line)
  infopen.close()
  outopen.close()
quchong('11.txt','22.txt')

```



