Shinobi
=========

[![Build Status](https://travis-ci.org/charliemaiors/shinobi.svg?branch=master)](https://travis-ci.org/charliemaiors/shinobi)

Shinobi role will deploy [Shinobi CCTV](https://shinobi.video/) system Ubuntu, CentOS or Archlinux systems. Using variables is possible to deploy the CE or the Pro version.

Requirements
------------

The role requirements are [Nodejs with npm](https://galaxy.ansible.com/charliemaiors/nodejs) and [FFMPEG](https://galaxy.ansible.com/charliemaiors/ffmpeg); it requires also an instance of Mariadb/Postgres up and running
with shinobi database and schemas already created, or can be deployed using [shinobi-db role](https://galaxy.ansible.com/charliemaiors/shinobi_db).

Role Variables
--------------

This role requires three main variables:

```yaml
db_host: "localhost"
shinobi_pass: "test"
shinobi_port: "8080"
```

The ```db_host``` variable is the address of shinobi database, the role assumes that is already created and configured; if the address is wrong the role WILL NOT fail but Shinobi will not work.
The ```shinobi_pass``` variable is the password for the shinobi backend service in order to connect to the database.
The ```shinobi_port``` variable is the port 
Default variables are:

```yaml
random_key: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters') }}"
ce_version: "https://gitlab.com/Shinobi-Systems/ShinobiCE.git"
pro_version: "https://gitlab.com/Shinobi-Systems/Shinobi.git"
ce: true
pro: false
lts: true
startup: true
shinobi_user: "shinobi"
shinobi_version: "HEAD"
dbhash: "md5"
```

The ```random_key``` variable is used for random password generation. The ```ce/pro_version``` are references for Shinobi Code repository for community and pro version respectively. The ```ce/pro``` flags defines which version download. The ```lts``` flag represent which version of node is installed in order to avoid to install sqlite dependencies (for v8 of nodejs sqlite installation from npm fails).
The ```shinobi_user``` is the default shinobi user for database, ```dbhash``` is the hashing algorithm used for password store (see [Depdendencies section](#Dependencies)).
The ```shinobi_version``` is the commit hash or branch which has to be cloned from shionbi repository.

Dependencies
------------

This role has a direct dependency with nodejs and ffmpeg roles and an indirect dependency with Mysql/Mariadb and shinobit schema defined. The direct dependencies list is:

* charliemaiors.nodejs (lts flag is applied also to nodejs role in order to determine which version install)
* charliemaiors.ffmpeg

The indirect dependency is with charliemaiors.shinobi-db using ```shinobi_user```, ```shinobi_pass``` as common variables and ```db_host``` variable could be determined using db host machine address (or could be also ```localhost``` in case of same machine).W

Example Playbook
----------------

This is an example with an all-in-one installation:

```yaml
  - name: Deploy Shinobi on Single host
  hosts: shinobi-all-in-one
  vars:
    shinobi_pass: "shinobi-test-machine"
    db_host: "localhost"
    shinobi_port: "8080"
    user_pass: "shinobi-test"
    user_mail: "ccio@m03a.ca"
    dbhash: "sha256"
    startup: true
    ce: false
    pro: true
    lts: true
  roles:
    - charliemaiors.nodejs
    - charliemaiors.ffmpeg
    - charliemaiors.shinobi-db
    - charliemaiors.shinobi
```

This is an example with database on a different machine:

```yaml
- name: Deploy shinobi db
  hosts: shinobi-db
  vars:
    shinobi_pass: "shinobi-test-machine"
    user_pass: "shinobi-test"
    user_mail: "ccio@m03.ca"
  roles:
    - charliemaiors.shinobi-db

- name: Deploy shinobi frontend
  hosts: shinobi-fe
  vars:
    db_host: "{{ hostvars[groups['shinobi-db'][0]].ansible_host }}" # db host machine ip
    shinobi_port: "8080"
    user_pass: "shinobi-test"
    user_mail: "ccio@m03.ca"
    shinobi_pass: "shinobi-test-machine"
    startup: true
    ce: false
    pro: true
  roles:
    - charliemaiors.nodejs
    - charliemaiors.ffmpeg
    - charliemaiors.shinobi
```

License
-------

GNU GPL

Author Information
------------------

This role was created in 2018 by Carlo Maiorano as developer for Dipartimento di Informatica - Scienza e Ingegneria of Alma Mater Studiorum directed and supervisioned by Paolo Bellavista as Group Leader.