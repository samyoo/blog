---
title: 发布release命令
date: 2018-03-01 11:33:44
tags: [linux,ubuntu]
categories: 运维
---

### 在`/usr/local/bin/` 目录下新建`release`文件 受权755

```shell
#!/bin/sh

## create by sam
## release the projects

DATE=$(date +%Y%m%d)
INPUT=$1
case $INPUT in
	'esr-sdap'|'tracker'|'tracker-job'|'sdap')
		if [ ! -d ~/$INPUT ];then
       		    echo 'not project release'
		    exit 1
		fi
		cd /opt/tomcat/webapps/
		DIR='/home/sdap/bak/'$INPUT'_bak_'$DATE 
		if [ -d $DIR ]; then
		    DIR=$DIR'_'$(date +%H%M%S);
		fi
		echo 'backup to : '${DIR}
		mv ./${INPUT} $DIR 
		mv ~/${INPUT} ./
		cp -r ${DIR}/upload ./${INPUT}/
		cp ${DIR}/WEB-INF/classes/app-config.txt ./${INPUT}/WEB-INF/classes/
		cp ${DIR}/WEB-INF/classes/job.properties ./${INPUT}/WEB-INF/classes/
		;;
	'start')
		cd /opt/tomcat/bin/
	        #./catalina.sh start
		sh ./startup.sh
                #sleep 1;
                #tail -fn 200 ../logs/catalina.out

		;;
	'stop')
		ps -ef |grep tomcat |grep -v grep |awk '{print $2}'| xargs kill -15
		echo 'kill the tomcat.'
		;;
	*)
		echo "Usage: release esr-sdap|tracker|tracker-job|sdap|start|stop" 
		exit 1
                ;;

esac

#autotab_list=("esr-sdap" "tracker" "start" "stop")
#function _release() {
#    local cur="${COMP_WORDS[COMP_CWORD]}"
#    COMPREPLY=( $(compgen -W "${autotab_list[*]}" -- ${cur}) )
#}
#complete -F _release release 

#_foo()  
#{  
#    local cur=${COMP_WORDS[COMP_CWORD]}  
#    COMPREPLY=( $(compgen -W "exec help test" -- $cur) )  
#}  
#complete -F _foo foo
exit 0


```


### 命令自动补, 在`/etc/bash_completion.d`目录下添加文件`release.bash` 内容如下

```shell
autotab_list=("esr-sdap" "tracker" "tracker-job" "sdap"  "start" "stop")
function _release() {
    local cur="${COMP_WORDS[COMP_CWORD]}"
    COMPREPLY=( $(compgen -W "${autotab_list[*]}" -- ${cur}) )
}
complete -F _release release


```