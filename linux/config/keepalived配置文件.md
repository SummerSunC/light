```

global_defs {
 router_id TFS_01
}
vrrp_script check_run {
   script "/etc/keepalived/check_tfs"
   interval 5
}
vrrp_sync_group VG1 {
    group {
          VI_TFS
    }
}
vrrp_instance VI_TFS{
    state BACKUP
    nopreempt
    interface eth0
    virtual_router_id 85
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass tfs001
    }
    track_script {
        check_run
    }
    virtual_ipaddress {
        10.129.195.71/24 dev eth0 label eth0:0
    }
    notify_master /etc/keepalived/tfs_master.sh
    notify_backup /etc/keepalived/tfs_backup.sh
}
```