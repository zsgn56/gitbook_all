1、读取所有project信息

```py
#!/usr/bin/env python
#-*-coding:utf-8-*-
import requests
url = 'http://gitlab-ee22.cn/api/v4/projects?private_token=cLvSasxK1BmFyWuL_zNm&per_page=500'
user_url= 'http://gitlab-ee22.cn/api/v4/users?private_token=cLvSasxK1BmFyWuL_zNm&per_page=500'
#获取项目id和项目名称
def GetProject_id(project_url):
    r = requests.get(project_url)
    data = r.json()
    ProjectId_list = []
    ProjectName_list = []
    for i in data:
        ProjectId_list.append(i['id'])
        ProjectName_list.append(i['name'])
    return ProjectId_list,ProjectName_list
#根据项目id获取项目下的用户信息
def GetProject_userlist():
    IdList = GetProject_id(url)
    project_id = IdList[0]
    project_name = IdList[1]
    for id in project_id:
        l = []
        print id
        project_user = requests.get(user_url.format(id))
        #生成完整的用于显示项目下所有user的连接
        req_data = project_user.json()
        for i in req_data:
            l.append(i['name'])
        print (project_name[project_id.index(id)],l)
GetProject_userlist()
```

2、根据项目id批量删除

```py
#!/bin/env python
# -*- coding: utf-8 -*-

import requests

base_url = "http://gitlab-ee22.cn/api/v4/projects/"

headers = {
    'private-token': "cLvSasxK1BmFyWuL_zNm"
}

projects = [
"218"
]



for project in projects:
    url = base_url + project
    response = requests.request("DELETE", url, headers=headers, verify=False)
    print(response.text)
```

3、列出所有用户信息，v3 api

```py
#!/usr/bin/env python
#-*-coding:utf-8-*-
import requests
user3_url='https://gitlab.homeking365.com/api/v3/users?private_token=sNBuM8kyzZh62GishdnD&per_page=500'
def GetUserlist():
    r = requests.get(user3_url)
    data = r.json()
    Name_list = []
    UserName_list = []
    for i in data:
        Name_list.append(i['name'])
        UserName_list.append(i['username'])
    print i['name'],i['username'],i['state']
GetUserlist()
```



