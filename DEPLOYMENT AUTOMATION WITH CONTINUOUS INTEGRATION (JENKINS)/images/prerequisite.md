## PREPARE THE WEB AND NFS SERVER


NFS SERVER PREP (RHEL)
Install NFS Server
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```
```
sudo mkdir -p /mnt/apps
sudo chown -R nobody: /mnt/apps
sudo chmod -R 777 /mnt/apps
```
`sudo systemctl restart nfs-server.service`

Make directory accessible for all in the subnet CIDR
 `sudo vi /etc/exports`
`/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

**Important note:** In order for NFS server to be accessible from your client(webservers), you must also open following ports: TCP 111, UDP 111, UDP 2049


WEB SERVER PREP (RHEL)<br>
Install NFS client<br>
`sudo yum install nfs-utils nfs4-acl-tools -y`<br>
Mount /var/www/ and target the NFS server’s export for apps
`sudo mkdir /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`<br>
Verify that NFS was mounted successfully by running df -h. <br>
Make sure that the changes will persist on Web Server after reboot:<br>
`sudo vi /etc/fstab`<br>
add following line
`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`<br>






