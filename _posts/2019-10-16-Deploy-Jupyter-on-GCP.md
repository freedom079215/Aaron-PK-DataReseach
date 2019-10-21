---
layout: post
title: Deploy Jupyter on GCP
date: 2019-09-26
excerpt: ""
tags: python, IDE
comments: true
---
# Deploy Jupyter on GCP

1. Create a new APP engine
2. Set static IP
3. Install required package through SSH shell
```python=
curl -O https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh
sha256sum Anaconda3-5.0.1-Linux-x86_64.sh
bash Anaconda3-5.0.1-Linux-x86_64.sh
source ~/.bashrc
conda list
```
* 手動加入path
```python=
~$ vim ~/.bashrc # 利用 Vim 編輯 .bashrc 檔

# 加在最後一行
export PATH=/home/davidtseng/anaconda3/bin:$PATH

# 再執行
~$ source ~/.bashrc
```

4. Install jupyter Notebook
5. check Jupyter Config
```python=
ls ~/.jupyter/jupyter_notebook_config.py
```

6. Set up the firewall rules
- Allow ports
- 

7. Keep permanent running
> 承載 Cloud Shell 工作階段的虛擬機器執行個體並非永久分配給 Cloud Shell 工作階段，而是會在工作階段閒置一小時後就終止。執行個體終止後，您在 $HOME 以外做出的修改一律無效。
> 

gcloud init
gcloud compute ssh [instance-name]

different user name
sudo bash (change to root)
su [username]


* use "nohup"
- check jobs
jobs -l

gcloud auth login [account]
gcloud config set project [projectid]


---
* check disk
```
- lsblk

- sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

sudo mkdir -p /mnt/disks/[MNT_DIR]

sudo mount -o discard,defaults /dev/[DEVICE_ID] /mnt/disks/[MNT_DIR]

sudo chmod a+w /mnt/disks/
```
- 既有硬碟調整

sudo df -h

1. resize on UI
2. Command
```
sudo growpart /dev/[DEVICE_ID] [PARTITION_NUMBER]

sudo resize2fs /dev/sda1
```

* set jupyter password
```python=
jupyter notebook password
```