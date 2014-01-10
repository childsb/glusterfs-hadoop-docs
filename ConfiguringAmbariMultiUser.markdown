**Multiple Users in Ambari**

* Stop the hadoop services

* Run this script on the node:

`#!/bin/sh`

`          gluster_mount=/mnt/glusterfs`
`          process_user=yarn`
`          process_group=hadoop`
`          yarn_nodemanager_remote_app_log_dir="/tmp/logs"`
`          mapreduce_jobhistory_intermediate_done_dir="/mr_history/tmp"`
`          mapreduce_jobhistory_done_dir="/mr_history/done"`
`          mapreduce_jobhistory_apps_logs="/app-logs"`
`          yarn_staging_dir=/job-staging-yarn/`
`          task_controler=/usr/lib/hadoop-yarn/bin/container-executor`
`          task_cfg=/etc/hadoop/conf/container-executor.cfg`


`          setPerms(){`
`            Paths=("${!1}")`
`            Perms=("${!2}")`
`            Root_Path=$3`
`            User=$4`
`            Group=$5`

`            for (( i=0 ; i<${#Paths[@]} ; i++ ))`
`             do`
`              mkdir -p ${Root_Path}/${Paths[$i]}`
`              chown ${User}:${Group}  ${Root_Path}/${Paths[$i]}` 
`              chmod ${Perms[$i]} ${Root_Path}/${Paths[$i]}` 
`              echo ${Paths[$i]} ${Perms[$i]}`
`             done` 
`          }`

`          echo "Setting permissions on Gluster Volume located at ${gluster_mount}"`

`          paths=("/tmp" "/user" "/mr_history" "${yarn_nodemanager_remote_app_log_dir}"    "${mapreduce_jobhistory_intermediate_done_dir}" "${mapreduce_jobhistory_done_dir}" "/mapred" "${yarn_staging_dir}" "${mapreduce_jobhistory_apps_logs}");`
`          perms=(1777 0775 0755 1777 1777 0750 0770 0770 1777);`
`          setPerms paths[@] perms[@] ${gluster_mount} ${process_user} ${process_group}`

`          echo "Setuid bit on task controller"`
`          chown root:${process_group} ${task_controler} ; chmod 6050 ${task_controler}`
`          chown root:${process_group} ${task_cfg}`

* modify my /etc/hadoop/conf/container-executor to look like:

`          yarn.nodemanager.linux-container-executor.group=hadoop `
`          banned.users=yarn min.user.id=1000 `
`          allowed.system.users=tom`