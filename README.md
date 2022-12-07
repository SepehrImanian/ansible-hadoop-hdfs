# Ansible Playbook For Setup Hadoop HDFS

## Roles Variables

### Installation Vars
```yaml
hdfs_unarchived_filename: "hadoop-3.3.3"
hdfs_download_link: "https://downloads.apache.org/hadoop/common/{{hdfs_unarchived_filename}}/{{hdfs_unarchived_filename}}.tar.gz"
hdfs_download_sha_sum: "9ac5a5a8d29de4d2edfb5e554c178b04863375c5644d6fea1f6464ab4a7e22a50a6c43253ea348edbd114fc534dcde5bdd2826007e24b2a6b0ce0d704c5b4f5b"
hdfs_download_destination: "/opt/{{hdfs_unarchived_filename}}.tar.gz"
hdfs_unarchive_destination: "/opt/"
```

### File core-site.xml Vars
```yaml
hdfs_fs_defaultFS: "ha-cluster"
hdfs_trash_interval: 1440
hdfs_dfs_journalnode_edits_dir: "/var/lib/hadoop/journal"
hdfs_ha_zookeeper_quorum:
  - "172.16.21.125:49162"
  - "172.16.21.125:49163"
  - "172.16.21.125:49164"
```

### File hadoop-env.sh Vars
```yaml
hdfs_namenode_user: root
hdfs_datanode_user: root
hdfs_secondarynamenode_user: root
hdfs_journalnode_user: root
hdfs_zkfc_user: root
hdfs_java_home: /usr
hdfs_hadoop_home: /opt/hadoop
hdfs_hadoop_conf_dir: /opt/hadoop/etc/hadoop
hdfs_hadoop_daemon_root_logger: WARN,RFA
hdfs_hadoop_heapsize_max: ''
hdfs_hadoop_heapsize_min: ''
hdfs_hadoop_jaas_debug: ''
hdfs_hadoop_workers: ''
hdfs_hadoop_log_dir: ''
hdfs_hadoop_pid_dir: ''
hdfs_hadoop_niceness: ''
```

### File hdfs-site.xml Vars
```yaml
hdfs_dfs_replication: 1
hdfs_dfs_namenode_name_dir: /var/lib/hadoop/hadoop-name-{{hostvars[inventory_hostname]['name']}}
hdfs_dfs_nameservices: "ha-cluster"
hdfs_dfs_datanode_data_dir: /var/lib/hadoop/hadoop-data-{{hostvars[inventory_hostname]['name']}}
hdfs_ha_automatic_failover_enabled: "true"
hdfs_dfs_ha_fencing_methods: "sshfence"
hdfs_dfs_ha_fencing_ssh_private_key_files: /root/.ssh/id_rsa
hdfs_dfs_namenode_datanode_registration_ip_hostname_check: ''
```

### Disk Management Vars
```yaml 
disk: /dev/sdb
destination: /var/lib/hadoop
filesystem: ext4
allow_disk_mount: false
```

### Dns Vars

```yaml
dns_server: 172.16.21.128
allow_dns: true
```

## Example Playbook

### Inventory

```ini
[activenamenodeserver]
172.16.21.125 name=n1 rpc_port=9000 http_port=9870

[standbynamenodeservers]
172.16.21.127 name=n2 rpc_port=9000 http_port=9870

[namenodeservers:children]
activenamenodeserver
standbynamenodeservers

[datanodeservers]
172.16.21.128 name=n3
172.16.21.129 name=n4

[qjournalnodeservers]
172.16.21.125
172.16.21.127
172.16.21.128

[all:vars]
ansible_user=root
ansible_connection=ssh
ansible_ssh_port=22
```

### Installation

```yaml
- name: install-hdfs
  hosts: all
  become: true
  become_user: root
  roles:
    - install-hdfs

- name: start journalnodes
  hosts: qjournalnodeservers
  roles:
    - journalnode

- name: start namenodes
  hosts: namenodeservers
  roles:
    - namenode

- name: start datanodes
  hosts: datanodeservers
  roles:
    - datanode
```
```
ansible-playbook -i inventory.ini hdfs.yaml
```

### Destroy Whole HDFS After Installation

```yaml
- name: destroy-hdfs
  hosts: all
  become: true
  become_user: root
  strategy: linear
  roles:
    - destroy-hdfs
```
```
ansible-playbook -i inventory.ini destroy.yaml
```
