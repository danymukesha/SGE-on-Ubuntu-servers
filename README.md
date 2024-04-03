# SGE-on-Ubuntu-servers
Tutorial for setting up SGE on Ubuntu servers, both as master and client nodes. 
It covers installation steps and configurations required for proper functioning of the Grid Engine system.


**Setting up SGE (Sun Grid Engine) on Ubuntu**

**Master Server - Part 1**

1. **Update:**  
   ```
   apt-get update
   ```

2. **Add a new user:**  
   ```
   adduser gsadmin --uid 500
   ```

3. **Download Grid Engine package:**  
   ```
   wget http://downloads.sourceforge.net/project/gridscheduler/GE2011.11p1/GE2011.11p1.tar.gz
   tar -zxvf GE2011.11p1.tar.gz
   mv GE2011.11p1 /home/gsadmin/
   chown -R gsadmin:gsadmin /home/gsadmin/
   ```

4. **Install NFS Server (Network File System):**  
   ```
   apt-get install nfs-kernel-server
   echo "/home/gsadmin *(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports
   exportfs -a
   service nfs-kernel-server restart
   ```

5. **Install OpenJDK-8:**  
   ```
   apt-get install openjdk-8-jdk
   echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/" >> ~/.bashrc
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
   echo "export PATH=$PATH:/usr/lib/jvm/java-8-openjdk-amd64/bin" >> ~/.bashrc
   export PATH=$PATH:/usr/lib/jvm/java-8-openjdk-amd64/bin
   echo "export SGE_ROOT=/home/gsadmin/GE2011.11p1" >> ~/.bashrc
   export SGE_ROOT=/home/gsadmin/GE2011.11p1/
   source ~/.bashrc
   ```

**Host (Client) Server - Part 1**

1. **Add a user:**  
   ```
   adduser gsadmin --uid 500
   ```

2. **Install NFS client:**  
   ```
   apt-get install nfs-common
   ```

3. **Mount shared directory:**  
   ```
   mount xxx.xxx.xxx.xxx:/home/gsadmin /home/gsadmin/
   # Note: Handle any errors if encountered
   ```

4. **Set up auto-mount:**  
   ```
   echo "xxx.xxx.xxx.xxx:/home/gsadmin /home/gsadmin nfs" >> /etc/fstab
   ```

**Master Server - Part 2**

1. **Install gridengine-master:**  
   ```
   apt-get install gridengine-master
   ```

2. **Modify /etc/hosts:**  
   ```
   #127.0.0.1
   #127.0.1.1
   xxx.xxx.xxx.xxx master master
   yyy.yyy.yyy.yyy node01 node01
   zzz.zzz.zzz.zzz node02 node02
   ```

3. **Restart after modifications:**  
   ```
   service gridengine-master restart
   ```

4. **Confirm installation:**  
   ```
   qhost
   # Ensure qmaster entry in /etc/hosts and /var/lib/gridengine/default/common/act_qmaster match
   ln -s /var/lib/gridengine/default /home/gsadmin/GE2011.11.pl/default
   ```

**Host (Client) Server - Part 2**

1. **Install gridengine-client:**  
   ```
   apt-get install gridengine-client
   ```

2. **Install gridengine-exec:**  
   ```
   apt-get install gridengine-exec
   ```

3. **Restart:**  
   ```
   /etc/init.d/gridengine-exec restart
   ```

**Master Server - Part 3**

1. **Add nodes to admin list:**  
   ```
   qconf -ah node01
   qconf -ah node02
   ```

2. **Add gsadmin group:**  
   ```
   qconf -au gsadmin gsadmin
   ```

3. **Add nodes to commit hosts:**  
   ```
   qconf -as node01
   qconf -as node02
   ```

4. **Add queues to cluster:**  
   ```
   qconf -aq main.q
   ```

5. **Modify main.q:**  
   ```
   qconf -mq main.q
   # Configure hostlist and slots accordingly
   ```

6. **Restart:**  
   ```
   service gridengine-master restart
   ```

**Host (Client) Server - Part 3**

1. **Modify /etc/hosts to match master server:**  
   ```
   # Update /etc/hosts to match the master server
   ```

2. **Restart:**  
   ```
   /etc/init.d/gridengine-exec restart
   ```

3. **Check status:**  
   ```
   qhost
   ```

**End**

Additional commands such as `qstat`, `qsub`, `qmod`, etc., can be used for various tasks.

- `qstat -f`: To display full and detailed information about all jobs and queues.
- `qstat -F`: To display complete information about all jobs and queues.
- `qmod -c main.q@hostname`: To clear a job from a specific queue (`main.q`) at a given hostname.
- `qmod -d main.q@hostname`: To disable a job in a specific queue (`main.q`) at a given hostname.
- `qmod -e main.q@hostname`: To enable a job in a specific queue (`main.q`) at a given hostname.
- `qdel listnum`: To delete a job with the specified job number.
- `qsub`: To submit a job for execution.
