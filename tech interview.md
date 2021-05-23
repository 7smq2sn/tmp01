

# Networks

### OSI and TCP/IP models

OSI is a general, protocol independent model, which (kinda) includes TCP/IP, IPX/SPX, X.25 and other sets of protocols. It took place after TCP/IP. 
OSI is not confined to lan/internet-related communication, it also includes protocols like bluetooth and usb.

<img src="https://habrastorage.org/getpro/habr/comment_images/219/06c/99d/21906c99d593e1cbcef7d235df6b69e8.png" title="" alt="image" data-align="center">

### TCP/UDP

**TCP**
require handshake before any data being sent
offers retransmission and acknowledgment of the packets
slow but reliable data transmission

**UDP**
server sends packages right after request
every packet is independent, port / ip can be easily changed
checksum is available (used to verify data's integrity)
fast and unreliable

**how to 'ensure' udp packets, so that you can handle losses**:
create table on the receiving size, mark packets and keep track of them on the receiving side + requesting those which missing / not delivered yet 
or send ack for received packages

# 

# Linux

### permissions

`{a}rwxrwxrwx {b} owner   group  size ...`

`-rw-rw-r--   1   user01  adm    10    Apr 5 12:06 tmp01`

`{a}` - file type: 
    `-` regular file 
    `d` directory ("object" that "points" to files or other dirs)
    `l` symbolic link

owner (u) -> group (g) -> others (o) (not owner nor group's participants)

`{b}` - amount of hard links

`$ chmod` - change permissions (**only owner and root** can change permissions of files and directories)

`$ chmod a+rw tmp01` - rw for u+g+o

`$ chmod o+rw tmp01` - rw for others only

`dr-x...` allows to "read" directory, see the content 
`d-wx...` allows to "work" in directory (delete, create, rename files)

**r = 4, w = 2, x = 1**

`$ chmod 640 tmp01` -> -rw- r-- --- tmp01

### ownership

`$ chown` - change ownership

`$ chown [options] owner:group [file]` - can change either just owner, just group or both simultaneously

`$ chgrp [options] group [file]`

**file / directory ownership change**: 
    **owner**: can only change group to one that he's part of (by default)
    **root**: can change both owner and group

### groups

**Primary / login group** is the one that's assigned during user account creation (uid = gid, starting from 1000, by default), other groups, that user belongs to, are **secondary**, like adm, sambashare etc.

`usermod -a -G GROUP USER` - add user to a group `usermod` also able to change lot's of other user profile related stuff: name, shell, uid etc.

### su vs sudo

**su** stands for substitute user

with su you can: 
    **>** login as another user (root or regular user) within current terminal session 
    **>** execute commands as another user 
    **>** change shells within current terminal

you use **other user's password** to do these things

**sudo** stands for super or substitute user do

you can do same things as with su, but sudo grants access differently: it checks your account against `/etc/sudoers` file and if you pass the check (your account is part of adm or sudo group), then you prompted to enter **your password** to gain privileges or run a command

### links

**hard links**: 
    same inode (all links point to the same data on the drive)
    only for files within same disk / volume
    change either file or link -> all hard links + orig file change 
`$ ln [original file] [link name]`

**symbolic / soft links**: 
    unique inode
    for files and dirs on any disk/volume
    more like a shortcut, delete original -> broken link 
`$ ln -s [original file/dir] [link name]`

**inode** contains metadata about file / directory: permissions, owners, size, date etc. and location of the actual data

### drives, partitions, volumes

**drive**: physical device, represented by special file system objects called device nodes which are visible under the `/dev` directory. ex: `/dev/sda`

**partition**: logically separated part of the drive

**volumes**: alternative to partitions (?), uses LVM (logical volume manager)

**drive formatting** - preparation for it to store data/files. **stages**: 
low level formatting: controller related things, done in factories
partitioning: creating separately manageable regions - partitions
high level formatting: setting up new file system on top of partitions or logical volumes (lvm thing)

**partition table** (MBR pt or GUID pt) stores info about partitions of the physical drive.

**filesystems** handle the data on the partition (storage, retrieval, modification etc.)

`fdisk` partition's start and end sectors, partition size

`df -h` (disk free) details only about mounted fs'(real + others), shows fs type, size, used, avail, mount point
Only the fs' that start with a `/dev` are actual devices or partitions.

`lsblk --output name,fstype,fssize,fsavail,fsused,fsuse%` useful info on fs'

### filesystem hierarchy

**symbolic links:** 
    /bin + /sbin -> /usr/bin + sbin
    /lib + 32 + 64 + x32 -> /usr/lib + 32 + 64 + x32

/bin - exec files

/etc - system-wide configs

/media - mount points for flash drives, cd's

/mnt - temporally mounted fs's, like win partition

/opt - for proprietary software

/home + /home/.configs - some configs there too, user-related ?

### environment variables

`printenv XDG_CURRENT_DESKTOP` or `echo $...`

`set` outputs env vars + shell vars, attribs etc.

`test_var01='test value 01'` - shell var

`export test_var01` - now it's temp env var, will disappear after current session

`export test_var01='test value 01'` - combo

**persistent env vars**:

`/etc/environment` system wide env vars `VAR_TEST="Test Var"`

`/etc/profile` system wide config for shell `export PATH=$PATH:$JAVA_HOME/bin` `export JAVA_HOME="/path/to/java/home"`

`/home/.bashrc` per shell user set up of command aliases and shell functions `export PATH="$HOME/bin:$PATH"` `source ~/.bashrc` will load new env vars to current session

### process management

`top`, `htop`, `ps aux`, `ps -ef`

you can kill a process from within `htop` with f9 + choose a term signal

`pgrep bash` find out process's pid

**ppid**: process's parent id, the process who spawned the other process

`$ kill {-15 or -TERM} [pid]` - gracefully stop a process

`$ kill {-9 or -KILL} [pid]` - kill a process

`$ killall [process name]` - `-15` to every instance of the process

`$ pkill {-15 or -9} [process name]` - terminate all firefox processes, more advanced than `killall`

**niceness** is the number value within -20 to 20 range by which process execution priority gets controlled
-20 -> less nice -> requires more resources
majority of the processes have 0 nice value

**zombie process**: completed process, but waiting for it's parent process to pick up the return value

### software installation and handling

software comes in form of **packages** (.rpm, .deb) or tar balls (.tar.gz, .tar.bz2 etc.)

**repositories** store these packages remotely

there's also Personal Package Archive (PPA) network for "individual" packages

**APT** (Advanced Package Tool) package manager gets repository info from here:
/etc/apt/sources.list
/etc/apt/sources.list.d/[files]

apt stores info on packages here:
/var/lib/apt/lists/

installing tar balls:
extract it with `tar` `$ ./configure` checks for libs and configs on your system `$ make` make installation instructions `$ make/check install` install the program from source code

packages usually get installed in `/usr/bin + /etc + /usr/lib`

`$ whereis firefox` firefox: /usr/bin/firefox /usr/lib/firefox /etc/firefox /usr/share/man/man1/firefox.1.g

### services and daemons

**daemons** are programs that run as a background process, they monitor system, run scheduled tasks via `cron` etc.

`$ systemctl --type=service --state=running`

`$ service cron status` works + auto completion

`$ service cron.service status` doesn't work

`$ systemctl status {cron or cron.service}` works + auto completion

`$ systemctl start/stop/restart/reload/status/enable/disable [service]`

reload: force the service to reload its configuration files
enable/disable to start when system boots

`httpd` is a daemon of Apache HTTP Server

### security

**Security-Enhanced Linux** is a set of kernel modifications and user-space tools that gives more control over access permissions between apps, processes, files via security policies.

**Access Control Lists** (ACL) helps add permissions to files and directories without modifying owners and groups.

`# getfacl ./file.tmp`

`# setfacl -d -m u:accounting:rwx /accounting`

-d: all content of this dir will inherit this ACL permissions

-m or -x: modify or remove

`# setfacl -m u:kenny:r-x /accounting`

`# setfacl -m fred:- ./kenny`

`# setfacl -m g:accounting:- ./kenny`

### pipelines

**standard streams** provide communication between program and it's environment.

`stdin`, `stdout`, `stderr`

**file descriptor**: stdin is 0, stdout is 1, and stderr is 2

`|` - pipeline character

`2>&1` redirect stderr to stdout

`$ echo "hello there" >&2` stdout -> stderr

```bash
$ cu
Command 'cu' not found, but can be installed with:
$ sudo apt install cu
$ cu 2>/dev/null // output: nothing
$ cu 2>&1  // output: same as just '$ cu'
```

```bash
echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
// hyphen (-) means use content of stdin where hyphen is
```

### grep/awk/sed

`grep 'word' file1 file2 file3`

`grep 'string1 string2' filename`

`grep "/var/l" passwd`

**awk** is utility that runs commands written in awk programming language

`$ cat passwd | awk '/user/ {print}'`

`$ awk '/user/ {print}' passwd`

**sed** stream editor

`$ sed -n /box/p passwd` "box"

`$ sed -n "/var\/lib/p" passwd` "/var/lib"

### bash

```bash
#!/bin/bash
((sum=25+35)) # Add two numeric value
echo $sum
```

`$ bash tmp.sh` or `$ chmod a+x tmp.sh` + `$ ./tmp.sh`

`$#` - total number of args `$@` - all args together `$1` - 1st arg

[30 Bash Script Examples – Linux Hint](https://linuxhint.com/30_bash_script_examples/)

[25 Bash Script Examples | FOSS Linux](https://www.fosslinux.com/42541/bash-script-examples.htm)

# Git and GitHub

git is **distributed version control system** (DVCS), **everyone**(server, like github or local clients) **has the exact same repository**, so if github is down, you can keep pushing your changes to your local repo

**git vs svn**: svn(subversion) is centlalized vcs -> you push/pull from one remote place only

git uses **snapshots** approach: file changed -> new "version" will replace this file with the new one
others use **delta-based** approach: file changed -> keep original files + these changes (deltas)

git can be used **completely locally** (storing all changes you've made, reverting to certain states etc.)

**file states** in local repository: **untracked**: wasn't part of the last snapshot, you added completely new file in repository directory, and you haven't added it to the staging area -> git doesn't know about it **tracked**: git tracks it, can be: 
**unmodified** 
**modified**: file changed, but not added for commit
 **staged**: file marked as part of a commit via `$ git add`, added to "staging area"(.index folder)) 
**committed**: file's data committed(added) to the repository's DB in .git folder via `$ git commit` (HEAD)) then updated LR can be `$ git push` to remote repository, like github

When you run `$ git init` in a new or existing directory, Git creates the `.git` directory, which is where almost everything that Git stores and manipulates is located.

`$ git clone` makes **full copy of the repo** (copies files + .git folder (?)), thus you can rollback to any previous version this repo provide you with

**git starts tracking after first commit only**

If you modify a file after you run `$ git add`, you have to run `git add` again to stage the latest version of the file.

`$ git diff` gives you precise info on your changes in working directory and staging area.
It’s important to note that `$ git diff` by itself doesn’t show all changes made since your last commit — **only changes that are still unstaged**. If you’ve staged all of your changes, `$ git diff` will give you no output.

`$ git commit -m "fix benchmarks for speed"`

Every time you perform a commit, you’re recording a snapshot of your project that you can revert to or compare to later.

Committing without `$ git add` : add **-a** flag `$ git commit -a -m 'Add new benchmarks'`

`$ git rm [file-name]` removes the file from git and deletes from the drive `$ git rm --cached [file-name]` removes from git, keeps on the drive

`$ git reset HEAD` or `$ git restore --staged` to unstage file

`$ git checkout --` or `$ git restore` revert modified file to it's committed or staged version

**HEAD** is a pointer to current branch

`$ git checkout [branch name]` - switches to different branch, HEAD pointing to this branch

new commit -> new snapshot

a branch in Git is actually a simple file that contains the 40 character SHA-1 checksum of the commit it points to

`$ git merge` merges branches into one
fast forward case: commits have direct reachable ancestors, thus pointer is just moved forward, without conflicts
both branches still exist and point to the same commit
you merge to the branch you currently checked out to

`$ git reset --hard HEAD~1` revert to commit before merge

**merge commit**: commit from merging distinct branches

**merge conflict** occurs when several commits change same lines of the same files and git doesn't know how to handle it. Options: abort merge, merge only '1st' commit or only '2nd' commit etc., merge all changes
For all other types of merge conflicts(deleted file etc.), you must resolve the merge conflict in a local clone of the repository and push the change to your branch on GitHub.

**merge conflict with remote involved**: trying to push -> getting conflict -> pulling updated repo -> resolving conflict -> pushing again

To synchronize your work with a given remote, you run a `$ git fetch <remote>` command (in our case, `$ git fetch origin`). This command looks up which server “origin” is (in this case, it’s `git.ourcompany.com`), fetches any data from it that you don’t yet have, and updates your local database, moving your `origin/master` pointer to its new, more up-to-date position.

When you want to share a branch with the world, you need to push it up to a remote to which you have write access. `$ git push <remote> <branch>`

`$ git rebase` puts all branch's commits on top of another branch (sorta - ?)
doesn't "preserve" history - ?

`$ git pull` = `$ git fetch` + `$ git merge`

### GitHub

**forking**: copying the repo to your account so that you can push without worries

**topic branch**: short-term branch devoted to particular feature/issue

**behind commits**: not present in this branch, present in master

**ahead commits**: present on this branch, not present in master

**Checking-in** code means to upload code to main branch repository so that its administrator can review the code and finally update the project version. Additionally, **checking-out** code is the opposite which means to download a copy of code from the repository. For example, on GitHub, **cloning means checking-out** code and **pull request means checking-in** code.

# Databases

**Structured Query Language** (SQL) is a programming language for managing the data in **Database Management Systems** (DBMS).

**Relational DB** stores data in tables.

**Relational DBMS**: MySQL, MS SQL Server, PostgreSQL, SQLite

rows (строки) or tuples (кортежи) = data

columns = fields or attributes

table = relation

**schema** = described structure for tables, fields, relationships etc. in the db.

**Primary key** (PK, первичный ключ) uniquely identifies each record in a table.
PK must contain UNIQUE values, and cannot contain NULL values.
A table can have only ONE primary key
PK can consist of single or multiple columns (fields).

**Foreign key** (FK, внешний ключ) is the one that's PK in related table

**Normalization** is the process of structuring a database in accordance with a series of so-called **normal forms** in order to reduce data redundancy and improve data integrity.

**Denormalization** is a strategy used on a previously-normalized database to increase read performance at the expense of losing some write performance, by adding redundant copies of data or by grouping data.

**Relationships** are connections between tables in DB. `one to one`, `one to many`, `many to one`, `self referencing` self referencing: table with car companies where one company owns/owned by another one

**Commit** is a confirmed/approved by dbms update of a record in the db. You will see changes in the record only after commit has been done successfully.

`TRUNCATE TABLE [table name]` removes all rows from a table, but the table structure and its columns, constraints, indexes, and so on remain. `DELETE FROM [table name]` without `WHERE` does the same.

```sql
-- deletes only the entries by provided criteria 
DELETE FROM Customers WHERE CustomerName='Alfreds Futterkiste';
```

`DROP TABLE [table name]` removes table completely.

**constraint** is used to specify rules for the data in a table. `NOT NULL`, `UNIQUE`, `PK`, `FK`, `CHECK`, `DEFAULT`, `CREATE INDEX`

```sql
CREATE TABLE Persons (
 ID int NOT NULL,
 LastName varchar(255) NOT NULL,
 FirstName varchar(255),
 Age int,
 CHECK (Age>=18),
 City varchar(255) DEFAULT 'Sandnes',
 PhoneNumber NOT NULL UNIQUE,
 Address NOT NULL UNIQUE
);
```

```sql
CREATE TABLE Orders (
    OrderID int NOT NULL,
    OrderNumber int NOT NULL,
    PersonID int,
    PRIMARY KEY (OrderID),
    FOREIGN KEY (PersonID) REFERENCES Persons(PersonID)
);
```

---

```sql
-- how many orders each employee has
SELECT COUNT(OrderID) as 'Amount of orders', EmployeeID
FROM Orders
GROUP BY EmployeeID
ORDER BY COUNT(OrderID) DESC
```

**clauses** are keyword that limit the result of the query by providing condition to it `WHERE`, `HAVING`, `JOIN`

A `JOIN` clause is used to combine rows from two or more tables, based on a related column between them.

![SQL JOIN, JOIN Syntax, JOIN Differences, 3 tables  with Examples   Dofactory](https://www.dofactory.com/img/sql/sql-joins.png)

```sql
-- only clients with orders
-- (INNER JOIN)
SELECT Orders.OrderID, Customers.CustomerName, Orders.OrderDate
FROM Orders INNER JOIN Customers 
ON Orders.CustomerID=Customers.CustomerID;
```

```sql
-- all clients with mapped order id's for those who ordered something
-- (LEFT JOIN)
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers LEFT JOIN Orders 
ON Customers.CustomerID = Orders.CustomerID
ORDER BY Customers.CustomerName;
```

```sql
-- clients who don't have any orders 
-- (LEFT OUTER JOIN)
SELECT Customers.CustomerName as Customer, 
Customers.CustomerID as ID, 'no orders' as Orders
FROM Customers LEFT JOIN Orders
ON Customers.CustomerID=Orders.CustomerID
WHERE Orders.CustomerID IS NULL
ORDER BY Customers.CustomerName
```

```sql
-- (FULL OUTER JOIN)
SELECT Customers.CustomerName, Orders.OrderID
FROM Customers
FULL OUTER JOIN Orders 
ON Customers.CustomerID=Orders.CustomerID
ORDER BY Customers.CustomerName;
```

---

**NoSQL DB's** aka **non-relational DB's**

**Document databases** store data in documents similar to JSON objects. 
Representatives: MongoDB

**Key-value databases** 
values can be simple strings or complex json objects
Use cases: user preferences, caching (large amounts of data without complex queries)
Representatives: Redis, Amazon DynamoDB

![Diagram showing an example of data stored as keyvalue pairs in DynamoDB](https://d1.awsstatic.com/product-marketing/DynamoDB/PartitionKey.8dd0530a7f6d66d101f31de30db515564f4cf28a.png)

**Wide-column stores** store data in tables, rows, and dynamic/arbitrary-length columns. 
Use cases: storing IoT data and user profile data. 
Representatives: Cassandra, HBase

**Graph databases** store data in nodes and edges. Nodes typically store the data while edges store info about the relationships between the nodes. 
Use cases: patterns in social networks interactions, fraud detection, recommendation engines. 
Representatives: Neo4j, JanusGraph

---

![enter image description here](https://i.stack.imgur.com/AB37p.png)

![enter image description here](https://i.stack.imgur.com/tp0Xd.png)

Online Transaction Processing System (OLTP)
Online Analytical Processing System (OLAP)

# DevOps

**Development env before DevOps practices**:

\> Dev team develops functionality, security, bug fixes etc. in their own env.
\> Pass the build with new functionality to Ops team for deployment
\> Ops's test it in their own env.(diff. configurations, version of modules, libs etc.)
\> problems arise

**DevOps** is a methodology / set of practices which help software development cycle by increasing speed, reliability and security.

DevOps uses agile paradigm, where software "emerges" in iterations, which contain certain functionality.

# CI / CD / CD

**CI / CD** are approaches of frequently and reliably delivering apps to customers by introducing automation into the stages of app development.

Continuous Integration (CI)
Continuous Delivery (CD)
Continuous Deployment (CD)

| Environment / Tier Name                                     | Description                                                                                 |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Local                                                       | Developer's desktop/workstation                                                             |
| Development                                                 | Development server acting as a sandbox where unit testing may be performed by the developer |
| Integration                                                 | CI build target                                                                             |
| Testing / Test / QC (quality control) / Internal Acceptance | can be separate or part of integration env.                                                 |
| Staging / Stage / Pre-production                            | Mirror of production environment                                                            |
| Production / Live                                           | Serves end-users/clients                                                                    |

![What are the differences between continuous integration, continuous delivery, and continuous deployment?  Atlassian CI/CD](https://wac-cdn.atlassian.com/dam/jcr:b2a6d1a7-1a60-4c77-aa30-f3eb675d6ad6/ci%20cd%20asset%20updates%20.007.png?cdnVersion=1551)

**CI tools**: Jenkins, Travis CI

---

# Jenkins

Jenkins is an automation server for building, testing and deploying

Written in Java, runs in servlet containers such as Apache Tomcat.

default port: 8080

**server restart**: localhost:8080/jenkins/restart

**Master -> Controller** - first and main node, stores configs, loads plugins etc.

**Agent** - a machine, which connects to a controller and executes tasks when directed by the controller.

**Node** - A machine which is part of the Jenkins environment and capable of executing Pipelines or Projects. Both the Controller and Agents are considered to be Nodes.

**Job -> Project** - a user-configured description of work which Jenkins should perform, such as building a piece of software, etc.

**Build** - result of a single execution of a Project
`/var/lib/jenkins/jobs/[project name]/builds`
**statuses**: aborted, failed, stable (successful + not unstable), successful (no errors), unstable (finished with errors)

**Publisher** - notifications/reports during or after build

**Artifact** - an immutable file generated during a Build or Pipeline run

**SCM** - source code management (git, subversion etc.)

**Apache Maven** -  build automation tool

**Selenium** - framework for testing apps

**Home directory** stores builds and archives. 
You can change the path with env. var JENKINS_HOME.

### github triggering

when you run a project with github repo in it, the project clones the repository

configuring trigger for a push event (lot's of options of events): 

- install github plugin

- project config -> build triggers -> "github hook trigger for gitscm polling" 
  
  \+ general -> github project

- configure webhook (notifies via POST the url you provide) on github repo
  payload url: ip:8080/github-webhook/
  content type: json

### script console

console runs commands via apache groovy language 

`println("test line")`

`"ls -la /etc".execute().text`

`Jenkins.instance.genNumExecutors()`- number of executors on controller (?)

### pipeline

need to install "pipeline" plugin

you code your pipeline in either `jenkinsfiles` (which you check in in project config) or directly in project config

you code pipeline using either declarative or scripted syntax.

**declarative**: less groovy dependent, simpler and more pre-defined structure -> less control over the script, support pipeline as a code model -> can commit to repository, revision the file etc.

**scripted**: groovy based -> more control over the script

```groovy
// Declarative pipeline
pipeline {
    agent {
        docker { // will create container on a node
            image 'node:lts-buster-slim'
            args '-p 3000:3000'
            label 'label01' // agent / node with this label
                            // will run the container
        }                   // without label it'll be controller 
    }
    environment {  // env vars
        CI = 'true' 
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') { 
            steps {
                sh './jenkins/scripts/test.sh' 
            }
        }
    }
}
```

```groovy
// Scripted pipeline
node { 
    stage('Build') { 
        sh 'make' 
    }
    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml' 
    }
    if (currentBuild.currentResult == 'SUCCESS') {
        stage('Deploy') {
            sh 'make publish'
        }
    }
}
```

---

# Terraform and Ansible

### Infrastructure as a Code (IaC)

IaC is the process of managing and provisioning cloud infrastructure through config files.

Approaches to IaC:
**Declarative** (functional) which defines the desired state and the system executes what needs to happen to achieve that desired state. 
**Imperative** (procedural) defines specific commands that need to be executed in the appropriate order to end with the desired conclusion.

Methods of IaC:
**push**: the server to be configured will pull its configuration from the controlling server
**pull**: the controlling server pushes the configuration to the destination system

### Terraform

Usage: Infra provisioning 
Approach: Declarative
Method: Push

**Provider** is a plugin that contains a collection of resources. TF installs this plugins so that it understand how to interpret resources.
Example: AWS provider which contains EC2, Lambda, EKS, ECS, VPC, S3, RDS, DynamoDB and other resources.

**Resource** is a block that describes one or more infrastructure objects.

```json
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9" // terraform version
}

provider "aws" {
  profile = "default"
  region  = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3" // amazon machine image (ubuntu)
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

`terraform init` - downloads and installs the providers (into `.terraform` dir) defined in the config

`terraform plan` - generates and outputs execution plan without performing it

`terraform apply` - generates execution plan and performs it
use it when changing the config as well

When you applied your configuration, Terraform wrote **state** data into a file called `terraform.tfstate`. Terraform stores the IDs and properties of the resources it manages in this file, so that it can update or destroy those resources going forward.
This state file is the only way Terraform can track which resources it manages.

`terraform show [file]` - outputs either state or plan file

`terraform destroy` - deletes only resources under management

**Importing existing infra to TF**
The current implementation of Terraform import can only import resources into the state. It does not generate configuration. 

Because of this, prior to running `terraform import` it is necessary to write manually a resource configuration block for the resource, to which the imported object will be mapped.

`$ terraform import aws_instance.foo i-abcd1234` - resource.name + id

**Module** 
is a set of config files in a single directory. Even a simple configuration consisting of a single directory with one or more `.tf` files is a module.

### Ansible

Usage: infra config (installs and manages software on existing servers) + provisioning
Approach: Imperative
Method: Push (sends tasks to the nodes of the infra)

Server connects to hosts via ssh, doesn't require agents, but needs installation of the ansible package

`/etc/ansible` - default working dir, can copy it anywhere and work there

`ansible.cfg` - parameters like env vars, location of inventory and library files etc. 

`hosts` - default **inventory** file, that describes **hosts** (machines to manage) and **groups** of hosts.

```yaml
# hosts example, can be ini or yaml
all: # default group
    hosts:
        mail.example.com:
    jumper:
        ansible_port: 5555 # alias
        ansible_host: 192.0.2.50
        # running Ansible against the host alias “jumper” 
        # will connect to 192.0.2.50 on port 5555
    children:
        webservers:
            hosts:
                foo.example.com:
                bar.example.com:
    atlanta:
        host1:
            http_port: 80 # vars can be used in playbooks
            maxRequestsPerChild: 808
          host2:
            http_port: 303
            maxRequestsPerChild: 909
```

`roles` dir contains other directories with roles

`roles/common/tasks/main.yml` - "common" is the name of the role

**Roles** are reusable sets of commands (copy something), tasks (install packages), var defs, handlers etc (each thing has it's own folder). which you can run against hosts or groups in a playbook.

```yml
# main.yml
- name: Install apache packages
      apt:
        name:
        - apache2
        update_cache: yes
        state: latest
```

**Playbooks** are files containing configs you want your host or group to have. Playbooks consist of **plays**. Plays are tasks that run against hosts. Each task calls an Ansible module.

```yml
# playbook example
---
- name: random name
  hosts: all # run against all hosts
  become: true # alternative to -b flag
  roles:
    - common # "roles/common" folder with the role
```

`$ ansible-playbook -K install-task.yml` - executes the playbook

`$ ansible -m ping all` - ping all hosts from the inventory

`$ ansible -m shell -a 'hostname' all` - run 'hostname' command on all hosts, will execute from current user
`-m` module to execute
`-a` module arguments

**Modules** are discrete units of code that can be used from the command line or in a playbook task (run a command on remote host, reboot a service etc.). Modules execute on the target system (usually that means on a remote system) in separate processes.

`$ ansible -b -K -m shell -a 'cat /etc/passwd' all`
`-b` run as root, without pass prompt (implies you don't need to type it)
`-K` ask for pass prompt

**Handler** is a task which is called only if a **notifier** is present.

**Plugins** are pieces of code that augment Ansible’s core functionality and execute on the control node within the `/usr/bin/ansible` process.

### Provisioning with Ansible

```yml
---
### provision AWS EC2 instance
- hosts: localhost
  gather_facts: False
  tasks:
    - name: Provision a set of instances
      ec2:
         key_name: my_key
         group: test
         instance_type: t2.micro
         image: "{{ ami_id }}"
         wait: true
         exact_count: 5
         count_tag:
            Name: Demo
         instance_tags:
            Name: Demo
      register: ec2
```

amazon.aws.ec2 module can create, terminate, start or stop an instance in ec2

```yaml
# Basic provisioning example
- amazon.aws.ec2:
    key_name: mykey
    instance_type: t2.micro
    image: ami-123456
    wait: yes
    group: webserver
    count: 3
    vpc_subnet_id: subnet-29e63245
    assign_public_ip: yes
```

The limitations, compared to Terraform, is that its imperative nature, meaning it goes top to bottom, if you missed / forgot to put in something earlier, like command or copy of a file somewhere, the whole process crashes whereas in TF the order in which you specify resources doesn't matter, TF will figure it out itself.

---

# Docker

**Docker daemon** (`dockerd`) listens for requests from the **client** and manages Docker objects such as images, containers, networks, and volumes.

**dockerfile** - a file that contains instructions on how to build an image

```dockerfile
FROM node:12-alpine # base image
RUN apk add --no-cache python g++ make 
# creates tmp container, executes "apk add", 
# transforms into read-only layer after that
WORKDIR /app # execution directory for RUN, CMD, COPY etc.
COPY . . # source -> destination (filesystem of the container)
RUN yarn install --production # install dependecncies from package.json
CMD ["node", "src/index.js"] # executed inside container, can be only one
```

**.dockerignore** - a file that contains patterns of files / dirs to exclude from sending to docker engine after `docker build` command

**image** is an immutable (unchangeable) file that contains the source code (like index.js to start nodejs server), libraries, dependencies, tools, and other files needed for an application or OS to run.
images are stored here: `/var/lib/docker/overlay2/[sha]/diff/..`

**parent / base image** is the image that your image is based on.
`FROM mongo:latest`

**base image** is an empty image, which used to create "base" images like ubuntu, mongo etc.
`FROM scratch`

**tags** - different version of images, can be `lts`, `stable`, `latest`, `0.1`, `1.1` etc.

**digests** are versions of the image for different architectures (amd64, arm etc.)

**Docker Hub** - public registry for images

**Container Registries** are catalogs of repositories
Docker Hub, Amazon ECR , Azure CR, Google CR

**Container Repositories** are collections of images

**container** - runnable instance of an image
Each container runs as an isolated process, has it's own filesystem, network, volumes 

### containers vs vm

containers handle requests faster than vm's (ranging from a bit to much faster depending on the actual load), while consuming less (cpu) or much less (ram, storage) resources

### why containers occupy less space then full OSes ?

containers contain userland (shell, syscalls etc.) and don't contain kernels, drivers etc.

<img src="https://jfrog--c.eu12.content.force.com/servlet/servlet.ImageServer?id=0151r000006uDeS&oid=00D20000000M3v0&lastMod=1584629589000" title="" alt="User-added image" data-align="center">

<img src="https://phoenixnap.com/kb/wp-content/uploads/2019/10/container-layers.png" title="" alt="Brief explanation of Container Layer and Image layer" data-align="center">

`$ docker build [options] PATH` - builds an image from a Dockerfile
`$ docker build -t app01:ver0.1 .` will look for the Dockerfile in the current dir

`$ docker images` list images

`$ docker run [image-name]` = `create` + `start` a container

`$ docker create [image-name]`
6d8af538ec541dd... (container's id)
`$ docker start [container-name or id]`

`$ docker run -p 8080:80 --name [container-name] -d [image]`
localhost:8080 -> container:80
name the container 
`-d` - run the container in the background

`$ docker run -p 3000:3000 --name cont01 -it --rm img01 `
`-i` : keep STDIN open even if not attached (without `-t` creates "kinda" interactive shell)
`-t` : allocate a pseudo-tty (this flag without `-i` creates "noninteractive" shell)
use `-it` for normal shell session
`--rm` : delete container when stopped

container's shell session exit without shutting down the container:
`docker run` with `-it`: ctrl + shift + q -> close the terminal window
`docker run` without `-it` and with `-d`: you can use `exit` when entering container with `$ docker exec -it cont01 "/bin/sh"` 

`$ docker ps` list running containers / processes
`$ docker ps -a` list running and stopped containers

`$ docker stop/start [container's name]`

`$ docker rm [container's name or id]`

### Volumes

Volume is a specially-designated directory within a container which is connected to the host. Volumes designed to persist data, independent of the container’s life cycle. 

**Named volume**
SQLite Database stores the data here: `/etc/todos/todo-app.db`
`$ docker volume create todo-vol01` (docker handles the volume itself)
We will use the named volume and mount it to `/etc/todos`, which will capture all files created at the path.
`$ docker run -p 3000:3000 -v todo-vol01:/etc/todos getting-started-app`

```bash
$ docker volume inspect todo-vol01
[
 {
 "CreatedAt": "2019-09-26T02:18:36Z",
 "Driver": "local",
 "Labels": {},
 "Mountpoint": "/var/lib/docker/volumes/todo-vol01/_data", 
    # actuall location of the data
 "Name": "todo-vol01",
 "Options": {},
 "Scope": "local"
 }
]
```

**Bind mounts**
you specify the exact mountpoint on the host. It's often used to provide additional data into containers (mount source code into the container to let it see code changes and respond to it).

```bash
docker run -dp 3000:3000 \
     -w /app \  # working dir inside container
     -v "$(pwd):/app" \ # bind mount the current directory 
     # from the host in the container into the /app directory 
     node:12-alpine \
     # yarn is a package manager for js
     sh -c """yarn install \ # install dependecncies from package.json
     && yarn run dev""" # runs "dev":"nodemon src/index.js"
```

### Multi-container apps

create network -> share it between containers

`$ docker network create todo-app-network`

```bash
 docker run -d \
     --network todo-app-network \
     --network-alias mysql \ # will help resolving ip address
     -v todo-mysql-data:/var/lib/mysql \ # mysql data store location
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \ # will create "todos" db
     mysql:5.7
```

```bash
docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app-network \
   -e MYSQL_HOST=mysql \ # makes use of --network-alias
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:12-alpine \
   sh -c "yarn install && yarn run dev"
```

### Docker compose

docker compose is a tool for running apps which use microservice architecture 
(1 container = 1 service / function) with whole config in one file `docker-compose.yml`

```yaml
version: "3.7"

services:
  app:         # first service, will have "app" network allias
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000  # host:container
    working_dir: /app
    volumes:
      - ./:/app # mount current dir on host to /app on container
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:         # second service, "mysql" network allias
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql 
        # volume doesn't get created automatically in "compose"
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

`$ docker-compose up`  will go through the config (building images, configuring and running containers)

`$ docker-compose down --volumes` stops and removes containers networks
can also delete: volumes and images, exception: `external` volumes and images

### Docker Swarm

Docker Swarm provides native clustering functionality for Docker containers, which lets you turn a group of Docker engines into a single, virtual Docker engine.

---

# Kubernetes (k8s)

**Kubernetes** is a cluster orchestration system for containers (runs containers in a cluster)

A **computer cluster** is a set of connected computers (nodes) that work together as if they are a single (much more powerful) machine. Unlike grid computers, where each node performs a different task, computer clusters assign the same task to each node.
Computer clustering relies on centralized management software that makes the nodes available as orchestrated servers.

**Kubernetes cluster** consists of two types of resources:

- **The Control Plane** (manages the cluster)

- **Nodes** (virtual or physical machines, which contain the services necessary to run **Pods**) 
  **node components**:
  
  - kubelet (agent that manages containers and their Pods) 
  
  - container runtime (responsible for running containers)
  
  - kube-proxy (maintains network communication)

![Components of Kubernetes](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

**Pod** is a group of one or more containers, with shared storage (volumes) and network resources, and a specification for how to run the containers.
Containers in a pod share an IP Address and port space.
Kubernetes manages Pods rather than managing the containers directly.

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a **Service**. Services allow your applications to receive traffic.

**Service** is a way to expose (enabling access to) an application running on a set of Pods.
Types: ClusterIP, NodePort, LB, ExternalName

<img src="https://miro.medium.com/max/374/1*7QXZHBty2KNFEtcE3MeIwA.png" title="" alt="Routing external traffic into your Kubernetes services | by Madeesha's Tech  Space | ITNEXT" data-align="center">

**workload** is an application running on Kubernetes

**Deployment** is a workload resource that provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state.
Deployment are versioned -> can rollback if something goes wrong.

**ReplicaSet** is a workload resource that ensures that predetermined number of replica Pods are running at any given time.

<img src="https://d33wubrfki0l68.cloudfront.net/7a13fe12acc9ea0728460c482c67e0eb31ff5303/2c8a7/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg" title="" alt="" data-align="center">

**Endpoint** is an object containing ip address and port of a pod. Pods get exposed to a service via endpoints.

**Rolling updates** allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.

`kubectl` is a command line tool for communicating with a Kubernetes API server.
You can use `kubectl` to create, inspect, update, and delete Kubernetes objects.
`ctl` stands for control

`kubeadm` is a tool for setting up a cluster.

### Helm

**Helm** is a package manager for Kubernetes

This "packages" contain preconfigured yaml files and called **Helm Charts**.

Also used as template engine, when you specify general config with placeholders and another config with rules for what exact values to put there instead.

---

---

---

# I / P / S aaS

<img title="" src="https://miro.medium.com/max/700/1*wMTNtI9E8Pk18-5wfCtmlQ.jpeg" alt="" data-align="inline" width="654">

**Platform as a Service**: you supply code, infra gets provisioned and managed automatically (some aspects can be configured)

**Function as a service** (FaaS): kinda like paas, but can and will be completely turned off if no load present whereas in paas case, a server always running. Used for non regular load, can be api calls, a web-based app etc.
Function is code you run within FaaS

# GCP

Google Compute Engine (GCE) (IaaS)
vm's

Google Kubernetes Engine (GKE)
Container as a Service (CaaS) / Kubernetes as a Service (KaaS)
Former Container Engine

Google App Engine (GAE)
PaaS
You need to handle only your code

Google Cloud Functions (GCF) — (FaaS)
allows one to write a simple function that when triggered, uses all the underlying infrastructure to compute and return a result.

---

# AWS

**EC2**: Elastic Compute Cloud (IaaS)
vm's

**ECS**: Elastic Container Service 
supports docker containers and lets you run applications on a managed cluster of EC2 instances. (CaaS)

**AMI**: Amazon Machine Image (Linux distros etc.)

**ARN**: amazon resource names - gives unique id's to aws resources

**IAM**: Identity and Access Management

**IAM roles** can give IAM users, groups or aws services temporal access to certain resources without changing their set of permissions

**SSM**: (simple) systems manager is an agent (on ec2) + server combo which you can use to run commands remotely 

**elastic beanstalk** is a PaaS

**lambda functions** is a FaaS

### Storage

**EBS**: elastic block storage - block-based storage
ebs volumes span one AZ and network attached to ec2 instances

**Instance stores** are physically attached non-persistent block-based storages to ec2's

**EFS**: elastic file system - file-based storage

**S3**: Simple Storage Service - object-based storage
management / communication is done via restful api (GET, PUT, POST etc.)
accessed via urls: s3.region.amazon.aws.com/bucket/object

### DB's

**RDS**: Relational Database Service
RDS supports such dbms / engines as mysql, ms sql server, postgresql

**Multi AZ** is a disaster recovery technology, where main RDS is in one AZ and **Standby copy** in another one. Copying happens synchronously: master waits conformation from standby copy that info is being written. MAZ is not available in free tier.

**Read replica** is used for scaling "reads": es2's can read from this replicas but they write to the master. Async copy, no conformation to master.

Amazon **DynamoDB** is a key-value nosql db

### Networks

**ENI**: Elastic network interfaces is a component that represents a virtual network card. 

**EIP**: elastic IP, static ip allocated for use from aws account

**route 53**: DNS service, health check of ec2's, redirecting traffic somewhere else if there're problems + redirection based on geo location, domain registration

**VPC**: virtual private cloud
a interconnect network of resource / services that can span several availability zones but confined to a region

**public subnet**: has access to the internet, one per AZ by default
**private subnet**: doesn't have access

**Security groups** are rules for incoming and outgoing traffic (firewall) for EC2 instances and other services, span a VPC. 
SG is a **stateful firewall** (if allowed in, won't check out)
support allow rules only
checks all rules before doing action

**Network ACL** (access control list) firewall on subnet level
NACL is a **stateless firewall** (checks both directions, no "sessions")
support allow and deny rules
applies first matching rule

`$ ssh -i [private key] ec2-user@[public dns or ip of the intsance]`

**ELB**: elastic load balancer
app level lb operates on http lvl
net level lb operates on tcp lvl

**Target group**: groups of instances or ip's where load will go

### CI/CD pipeline

**cloudFormation** is an infra as code, like terraform or ansible

With **CodePipeline** you can set up and configure pipeline, what repo to use (one created within aws via **CodeCommit** or github), build tools (jenkins or CodeBuilt) etc.

### Containers

**ECS**: Elastic Container Service

**Container definition** is the image for running the container

**Task definition** is config for container, like amount of cpu and memory, mapped ports

**Task** is a running container

**Service** keeps configured amount of tasks / containers running within it's environment,  also can loadbalance, autoscale (min / max amount of tasks)

**Cluster** is a logical grouping of tasks or services

**Fargate cluster** provisions resources and network (vpc, subnets) for your containers automatically you don't need to run anything yourself, but there's an option to run ec2 instance, install ECS on top of it, and run container from within it, it's called an ECS cluster

**EKS**: Elastic Kubernetes Service

`$ eksctl` - for creating clusters (needs authorization to work with aws)

---

# Security

### SSH keys

![](https://www.foxpass.com/hs-fs/hubfs/SSHkeydiagram.png?width=1539&name=SSHkeydiagram.png)

Keys being generated using RSA algorithm 

`$ ssh-keygen` - generates key pair

`$ ssh-copy-id [server]`

`$ ssh [server]` - now you can use these keys to establish connection

### Tokens

token - опознавательный значок / жетон

**access token** is a way to increase protection / privacy of a user

you log in to a server, receive a token for a session and if it gets stolen, it can be easily replaced contrary to your password which you can use elsewhere

there are several case, like 3rd part auth via facebook or google, where service operates via tokens only and never getting your pass from fb or google, where tokens are quite handy compared to traditional user+pass combination

# Python

```py
a_list = ['one', 'two', 'three', 4, 5] # list
tmp[0] --> 'one'
a_list.append(6)
a_list.remove('two')

a_dic = { # dictionary
    'key01':'value01', 
    'key02':'value02' 
} 
a_dic['key02'] --> 'value02'

a_set = { 1,1,1,2,2,3,3 } # set is a list with unique values only
a_set --> {1,2,3}

a_tuple = ('one','two') # tuples are immutable, used as dic keys

if (1 == 1) and ("dump stuff" == "dump stuff"):
    print('tmp msg')

for itme in a_list:
    print(item)

try:  # exceptions
    print(a_list[11])
except IndexError:
    print('item\'s not in the list')


def func01():
    print('tmp')

func01() --> tmp


class Person: # oop, class spawns objects
    def __init__(self): # constuctor
        print('new person')

per01 = Person() # becomes Person object
new person

import math # importing module / library, pip install [module]
print(math.pi) --> 3.14...

int(3.14) # convert float to int
float(314) # convert int to float
int("3.14") # convert str to int
```

# Windows

**Active Directory** (AD) is a database and set of services that contains information about the infrastructure, like user, computers, their permissions etc.

**Domain** is a logical group of related users, computers and other AD objects (shared folder, organizational unit, printer etc.)

domain -> tree -> forest

**Domain controller** (DC) is a server that's runs AD Domain Services (AD DS). DC responsible for authentication, granting access to resources in the domain.

AD uses LDAP, Kerberos, DNS

Org units (OU) can be comprised of users, groups, computers, printers, file shares etc.

Group policy is a feature

**Lightweight Directory Access Protocol** (LDAP) is application level (osi's 7th lvl) protocol that's used to communicate with AD, like HTTP, from what I understand.

**Access Control Lists** (ACL) is a set of rules that define which entities have which permissions on a specific AD object.

**Kerberos** is a network authentication protocol. It used at login, resources access etc.

**DNS**: conversion of ip to url (uniform resource locator)

# Other info

**Representational state transfer (REST)ful API** - uses HTTP's GET, PUT, POST, DELETE for handling the data

**scaling out / horizontally**: adding more components in parallel, more servers, more lb's etc.

**scaling up / vertically**: adding more power to a component so that it will withstand grater load, 1 cpu -> 2 cpu's, 4 -> 8 GB of memory on one vm

**SDLC**: software development life cycle is a process for planning, creating, testing, and deploying a system.

### ELK

<img src="https://www.itzgeek.com/wp-content/uploads/2020/06/Log-Monitoring-With-ELK-Stack.jpg" title="" alt="How To Install Elasticsearch, Logstash, and Kibana (ELK Stack) on Ubuntu  20.04" data-align="center">
