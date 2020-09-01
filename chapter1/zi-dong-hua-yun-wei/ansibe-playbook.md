范例集合：[https://github.com/kuailemy123/Ansible-roles](https://github.com/kuailemy123/Ansible-roles)

参考：

[http://blog.51cto.com/11134648/2157443](http://blog.51cto.com/11134648/2157443)

[https://www.w3cschool.cn/automate\_with\_ansible/automate\_with\_ansible-1rkd27pe.html](https://www.w3cschool.cn/automate_with_ansible/automate_with_ansible-1rkd27pe.html)

模块说明：

[https://docs.ansible.com/ansible/latest/modules/modules\_by\_category.html](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)

一、创建角色目录

```
mkdir /etc/ansible/roles/{角色名}
```

二、创建各个角色配置

1、supervisor

目录结构

```
├── defaults
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
└── templates
    ├── supervisord.conf.j2        #主配置文件
    ├── supervisor.service.j2      #centos7启动程序
    └── supervisor.sh.j2           #centos6启动程序
```

defaults/main.yml  定义基本配置

```
supervisor_conf_path: "/etc/supervisor"
supervisor_conf_project_path: "/etc/supervisor/conf"
supervisor_run_path: "/var/run/supervisor"
supervisor_log_path: "/var/log/supervisor"

supervisor_bin: "/usr/bin/supervisorctl"

supervisor_env: ""
supervisor_stopsignal: "TERM"
supervisor_program: []
# [{ name: 'superset', command: '/usr/local/bin/superset runserver', user: 'superset' }]
```

tasks/main.yml

```
- name: Check if supervisor local file is already configured.
  stat: path={{ supervisor_bin }}
  register: supervisor_bin_result

- name: Install pip.
  easy_install: name=supervisor

- name: Install supervisor for pip.
  pip: name=supervisor
  when: not supervisor_bin_result.stat.exists

- name: Create supervisor dir.
  file: dest={{ item }} state=directory mode=755
  with_items:
    - "{{ supervisor_conf_path }}"
    - "{{ supervisor_conf_project_path }}"
    - "{{ supervisor_run_path }}"
    - "{{ supervisor_log_path }}"

- name: Set supervisor config if configured.
  template:
    src: supervisord.conf.j2
    dest: "{{ supervisor_conf_path }}/supervisord.conf"

- block: 
  - name: Set supervisor start scripts if configured or centos6.
    template:
      src: supervisor.sh.j2
      dest: /etc/rc.d/init.d/supervisor
      mode: 755

  - name: Ensure supervisor is running for centos6.
    service: name=openresty state=started enabled=yes
  when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "6"

- block: 
  - name: Set supervisor start scripts if configured or centos7.
    template:
      src: supervisor.service.j2
      dest: /etc/systemd/system/supervisor.service

  - name: Ensure supervisor is running for centos7.
    systemd: name=supervisor state=started enabled=yes daemon_reload=yes
  when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
```

templates/supervisord.conf.j2

```
[unix_http_server]
file={{ supervisor_run_path }}/supervisord.sock

[supervisord]
pidfile={{ supervisor_run_path }}/supervisord.pid
logfile={{ supervisor_log_path }}/supervisord.log

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix://{{ supervisor_run_path }}/supervisord.sock

[include]
files = conf/*.ini


{% for p in supervisor_program %}
[program:{{ p.name }}]
process_name=%(program_name)s
command={{ p.command }}
directory={{ p.directory | d("/tmp") }}
stopsignal={{ supervisor_stopsignal }}
stopasgroup={{ p.stopasgroup | d("true") }}
killasgroup={{ p.killasgroup | d("true") }}
{% if p.user is defined %}
user={{ p.user }}
{% endif %}
{% if p.env is defined %}
environment={{ p.env }}
{% elif supervisor_env != "" %}
environment={{ supervisor_env }}
{% endif %}
startretries={{ p.startretries | d("3") }}
startsecs={{ p.startsecs | d("5") }}
stopwaitsecs={{ p.stopwaitsecs | d("10") }}
autostart={{ p.autostart | d("true") }}
autorestart={{ p.autorestart | d("true") }}
stdout_logfile_maxbytes={{ p.stdout_logfile_maxbytes | d("50MB") }}
stdout_logfile_backups={{ p.stdout_logfile_backups | d("10") }}
stderr_logfile_maxbytes={{ p.stderr_logfile_maxbytes | d("50MB") }}
stderr_logfile_backups={{ p.stderr_logfile_backups | d("10") }}
stdout_logfile={{ supervisor_log_path }}/{{ p.name }}-stdout.log
stderr_logfile={{ supervisor_log_path }}/{{ p.name }}-stderr.log
{% endfor %}
```

templates/supervisor.service.j2

```
# supervisord service for systemd (CentOS 7.0+)
#
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c {{ supervisor_conf_path }}/supervisord.conf
ExecStop=/usr/bin/supervisorctl -c {{ supervisor_conf_path }}/supervisord.conf shutdown
ExecReload=/usr/bin/supervisorctl -c {{supervisor_conf_path }}/supervisord.conf reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

templates/supervisor.sh.j2

```
#!/bin/bash
#
# supervisord   Startup script for the Supervisor process control system
#
# Author:       Mike McGrath <mmcgrath@redhat.com> (based off yumupdatesd)
#               Jason Koppe <jkoppe@indeed.com> adjusted to read sysconfig,
#                   use supervisord tools to start/stop, conditionally wait
#                   for child processes to shutdown, and startup later
#               Erwan Queffelec <erwan.queffelec@gmail.com>
#                   make script LSB-compliant
#
# chkconfig:    345 83 04
# description: Supervisor is a client/server system that allows \
#   its users to monitor and control a number of processes on \
#   UNIX-like operating systems.
# processname: supervisord
# config: {{ supervisor_conf_path }}/supervisord.conf
# config: /etc/sysconfig/supervisord
# pidfile: {{ supervisor_run_path }}/supervisord.pid
#
### BEGIN INIT INFO
# Provides: supervisord
# Required-Start: $all
# Required-Stop: $all
# Short-Description: start and stop Supervisor process control system
# Description: Supervisor is a client/server system that allows
#   its users to monitor and control a number of processes on
#   UNIX-like operating systems.
### END INIT INFO

# Source function library
. /etc/rc.d/init.d/functions

# Source system settings
if [ -f /etc/sysconfig/supervisord ]; then
    . /etc/sysconfig/supervisord
fi

# Path to the supervisorctl script, server binary,
# and short-form for messages.
supervisorctl=/usr/bin/supervisorctl
supervisord=${SUPERVISORD-/usr/bin/supervisord}
prog=supervisord
pidfile={{ supervisor_run_path }}/supervisord.pid
lockfile={{ supervisor_run_path }}/supervisord
STOP_TIMEOUT=${STOP_TIMEOUT-60}
OPTIONS="{{ supervisor_conf_path }}/supervisord.conf"
RETVAL=0

start() {
    echo -n $"Starting $prog: "
    daemon --pidfile=${pidfile} $supervisord $OPTIONS
    RETVAL=$?
    echo
    if [ $RETVAL -eq 0 ]; then
        touch ${lockfile}
        $supervisorctl $OPTIONS status
    fi
    return $RETVAL
}

stop() {
    echo -n $"Stopping $prog: "
    killproc -p ${pidfile} -d ${STOP_TIMEOUT} $supervisord
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && rm -rf ${lockfile} ${pidfile}
}

reload() {
    echo -n $"Reloading $prog: "
    LSB=1 killproc -p $pidfile $supervisord -HUP
    RETVAL=$?
    echo
    if [ $RETVAL -eq 7 ]; then
        failure $"$prog reload"
    else
        $supervisorctl $OPTIONS status
    fi
}

restart() {
    stop
    start
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    status)
        status -p ${pidfile} $supervisord
        RETVAL=$?
        [ $RETVAL -eq 0 ] && $supervisorctl $OPTIONS status
        ;;
    restart)
        restart
        ;;
    condrestart|try-restart)
        if status -p ${pidfile} $supervisord >&/dev/null; then
          stop
          start
        fi
        ;;
    force-reload|reload)
        reload
        ;;
    *)
        echo $"Usage: $prog {start|stop|restart|condrestart|try-restart|force-reload|reload}"
        RETVAL=2
esac

exit $RETVAL
```

2、filebeat

目录结构

```
├── defaults
│   └── main.yml
├── README.md
├── tasks
│   ├── configure.yml
│   ├── install.yml
│   └── main.yml
└── templates
    ├── filebeat.ini.j2
    ├── filebeat.init.j2
    └── filebeat.yml.j2
```

defaults/main.yml  定义基本配置

```
software_files_path: "/usr/loca/src"
software_install_path: "/usr/local"

filebeat_version: "6.5.4"

filebeat_file: "filebeat-{{ filebeat_version }}-linux-x86_64.tar.gz"
filebeat_file_path: "{{ software_files_path }}/{{ filebeat_file }}"
filebeat_file_url: "https://artifacts.elastic.co/downloads/beats/filebeat/{{ filebeat_file }}"

filebeat_pid_dir: "/var/run/filebeat"
filebeat_home_dir: "/usr/local/filebeat"
filebeat_log_dir: "/var/log/filebeat"
filebeat_supervisor_dir: "/etc/supervisor/conf"
filebeat_conf_dir: "{{ filebeat_home_dir }}/conf"
filebeat_data_dir: "{{ filebeat_home_dir }}/data"

filebeat_service_name: "filebeat"
filebeat_service_start: false
filebeat_conf_file: "{{ filebeat_service_name }}.yml"
filebeat_config : ""
```

tasks/main.yml

```
- include: install.yml
- include: configure.yml
```

tasks/install.yml

```
- name: install | Check if filebeat local file is already configured.
  stat: path={{ filebeat_file_path }}
  connection: local
  register: filebeat_file_result

- name: install | Create software files path.
  file: path={{ software_files_path }} state=directory
  connection: local
  when: not filebeat_file_result.stat.exists

- name: install | Download filebeat file.
  get_url: url={{ filebeat_file_url }} dest={{ software_files_path }} validate_certs=no
  connection: local
  when: not filebeat_file_result.stat.exists

- name: install | Confirm the existence of the installation directory.
  file: path={{ software_install_path }} state=directory

- name: install | Copy filebeat file to agent.
  unarchive:
    src: "{{ filebeat_file_path }}"
    dest: "{{ software_install_path }}"
    creates: "{{ filebeat_file | replace('.tar.gz','') }}"

- name: install | Check if filebeat remote soft link is already configured.
  stat: path={{ filebeat_home_dir }}
  register: filebeat_soft_link_result

- name: install | Create filebeat dir soft link.
  file: "src={{ filebeat_file | replace('.tar.gz','') }} dest={{ filebeat_home_dir }} state=link"
  when: not filebeat_soft_link_result.stat.exists
```

tasks/configure.yml

```
- name: configure | Create filebeat Directory.
  file: path={{ item }} state=directory
  with_items:
   - "{{ filebeat_pid_dir }}"
   - "{{ filebeat_log_dir }}"
   - "{{ filebeat_conf_dir }}"
   - "{{ filebeat_data_dir }}"

- name: configure | Setup filebeat yml file.
  template:
    dest: "{{ filebeat_conf_dir }}/{{ filebeat_conf_file }}"
    mode: 0644
    src: filebeat.yml.j2


- name: configure | Setup filebeat supervisor file.
  template:
    dest: "{{ filebeat_supervisor_dir }}/filebeat.ini"
    mode: 0644
    src: filebeat.ini.j2

- name: configure | Setup filebeat init file.
  template:
    dest: "/etc/init.d/{{ filebeat_service_name }}"
    mode: 0755
    src: filebeat.init.j2

- name: configure | Ensure filebeat is started and enabled on boot.
  service: "name={{ filebeat_service_name }} state=started enabled=yes"
  when: filebeat_service_start and filebeat_config

- name: configure | update supervisor config.
  shell: supervisorctl update

- name: configure | restart  supervisor.
  shell: supervisorctl restart filebeat
```

templates/filebeat.yml.j2

```
- type: log
  fields:
    # 区分类型,同logstash中解析对应
    type: Java-application
  paths:
    -  /logs/*/*-main.log
    -  /logs/*/*-request.log
  fields_under_root: true
  # include_lines: ["^ERR", "^WARN"] # 只发送包含这些字样的日志
  # exclude_lines: ["^OK"]           # 不发送包含这些字样的日志
  # document_type: "YII_dev_123_log" # 定义写入 ES 时的 _type 值
  # ignore_older: "24h"              # 超过 24 小时没更新内容的文件不再监听。在 windows 上另外有一个配置叫 force_close_files，只要文件名一变化立刻关闭文件句柄，保证文件可以被删除，缺陷是可能会有日志还没读完
  scan_frequency: "10s"           # 每 10 秒钟扫描一次目录，更新通配符匹配上的文件列表
  tail_files: false               # 是否从文件末尾开始读取
  harvester_buffer_size: 16384    # 实际读取文件时，每次读取 16384 字节
  backoff: "1s"                   # 每 1 秒检测一次文件是否有新的一行内容需要读取
  multiline:
    pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    negate: true
    match: after
```

templates/filebeat.init.j2  \#centos6下启动脚本

```
#!/bin/bash
#
# filebeat          filebeat shipper
#
# chkconfig: 2345 98 02
# description: Starts and stops a single filebeat instance on this system
#

### BEGIN INIT INFO
# Provides:          filebeat
# Required-Start:    $local_fs $network $syslog
# Required-Stop:     $local_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Filebeat sends log files to Logstash or directly to Elasticsearch.
# Description:       filebeat is a shipper part of the Elastic Beats 
#                     family. Please see: https://www.elastic.co/products/beats
### END INIT INFO



PATH=/usr/bin:/sbin:/bin:/usr/sbin
export PATH

[ -f /etc/sysconfig/filebeat ] && . /etc/sysconfig/filebeat

PIDFILE=${PIDFILE-/var/run/{{ filebeat_service_name }}.pid}
agent=${BEATS_AGENT-{{ software_install_path }}/filebeat/filebeat}
args="-c {{ filebeat_conf_dir }}/{{ filebeat_conf_file }} -path.home {{ software_install_path }}/filebeat -path.config {{ filebeat_conf_dir }} -path.data {{filebeat_data_dir }} -path.logs {{ filebeat_log_dir }}"
test_args="-e -configtest"
LOCKFILE="/var/lock/subsys/{{ filebeat_service_name }}"
LOGFILE="{{ filebeat_log_dir }}/{{ filebeat_service_name }}-wrapper.log"
RETVAL=0

CMD="$agent $args >> $LOGFILE 2>&1 & echo \$! > $PIDFILE"
# Source function library.
. /etc/rc.d/init.d/functions

test() {
    $agent $args $test_args >> $LOGFILE 2>&1
}

start() {
    echo -n $"Starting {{ filebeat_service_name }}: "
    test
    if [ $? -ne 0 ]; then
        echo "Exiting: error loading config file:{{ filebeat_conf_dir }}/{{ filebeat_conf_file }}"
     echo
    exit 1
    fi
    daemon --pidfile $PIDFILE $CMD
    RETVAL=$?
    [ $RETVAL -eq 0 ] && touch $LOCKFILE && success || failure
    echo
    return $RETVAL
}

stop() {
    echo -n $"Stopping {{ filebeat_service_name }}: "
    killproc -p $PIDFILE
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f $LOCKFILE $PIDFILE
    return $RETVAL
}

restart() {
    test
    if [ $? -ne 0 ]; then
    return 1
    fi
    stop
    start
}

case "$1" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    restart)
        restart
    ;;
    condrestart|try-restart)
        [ -f $LOCKFILE ] && restart || :
    ;;
    status)
        status -p $PIDFILE {{ filebeat_service_name }}
    RETVAL=$?
    ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart}"
        exit 1
esac

exit $RETVAL
```

templates/filebeat.ini.j2 加入到supervisor中

```
[program:filebeat]
environment=HOME=/usr/local/filebeat
directory=/usr/local/filebeat              ; filebeat 家目录
command=/usr/local/filebeat/filebeat -c /usr/local/filebeat/conf/filebeat.yml
numprocs=1
autostart=true
autorestart=true                           ;启动失败重试
startretries=5                             ;重启的次数
log_stdout=true                            ;日志标准输出
log_stderr=true
```



