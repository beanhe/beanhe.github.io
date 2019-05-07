---
layout: post
title: 使用JenkinsAPI创建Credential
category: jenkins
tags: [__beanhe,jenkins,ci,持续集成]
---
    
#### 使用Jenkins API创建Credential

###### 常用Jenkins Credential类别：

- 用户名/密码认证
- SSH认证

###### 通过API创建上述方式的credential

- 方式：用户名/密码，如图：

![](/files/201707/27155818946_11.png)

  - 代码如下：

```
curl -X POST 'http://<jenkins.server.com>/credentials/store/system/domain/_/createCredentials' --data-urlencode 'json={  
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "credentialid",
    "username": "apicredentials",
    "password": "P@$$W0rd",
    "description": "description",
    "stapler-class": "com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl"
  }
}'
```

- 方式：SSH密钥（直接指定），如图：

![](/files/201707/27160147712_22.png)

  - 代码如下：

```

curl -X POST 'http://<jenkins.server.com>/credentials/store/system/domain/_/createCredentials' --data-urlencode 'json={  
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "credentialid",
    "username": "apicredentials",
    "password": "",
    "privateKeySource": {
      "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey$DirectEntryPrivateKeySource",
      "privateKey": "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSUGPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XAt3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/EnmZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbxNrRFi9wrf+M7Q== schacon@mylaptop.local",
    },
    "description": "credential description",
    "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey"
  }
}'
```

- 方式：SSH密钥（密钥文件存在于master服务器上的指定位置），如图：

![](/files/201707/27160411035_33.png)

  - 代码如下：

```
curl -X POST 'http://<jenkins.server.com>/credentials/store/system/domain/_/createCredentials' --data-urlencode 'json={  
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "credentialid",
    "username": "apicredentials",
    "password": "",
    "privateKeySource": {
      "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey$FileOnMasterPrivateKeySource",
      "privateKeyFile": "/var/lib/jenkins/ssh.key",
    },
    "description": "credential description",
    "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey"
  }
}'
```

- 方式：SSH密钥（密钥文件存放在master服务器的~/.ssh目录），如图：

![](/files/201707/27160536694_44.png)

  - 代码如下：
  
```
 curl -X POST 'http://<jenkins.server.com>/credentials/store/system/domain/_/createCredentials' --data-urlencode 'json={  
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "credentialid",
    "username": "apicredentials",
    "password": "",
    "privateKeySource": {
      "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey$UsersPrivateKeySource",
    },
    "description": "credential description",
    "stapler-class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey"
  }
}'
 ```