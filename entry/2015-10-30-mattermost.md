---
layout: post
title: Mattermost安装
category: 协作工具
tags: [mattermost]
---
- Download

```
wget https://github.com/mattermost/platform/archive/v1.1.0.tar.gz
tar xzf mattermost.tar.gz
cd mattermost
```

- Pre config(config file and database with MySQL)

    - config file like this

    ```
    {
    "ServiceSettings": {
        "ListenAddress": ":8065",
        "MaximumLoginAttempts": 10,
        "SegmentDeveloperKey": "",
        "GoogleDeveloperKey": "",
        "EnableOAuthServiceProvider": false,
        "EnableIncomingWebhooks": true,
        "EnablePostUsernameOverride": false,
        "EnablePostIconOverride": false,
        "EnableTesting": false,
        "EnableSecurityFixAlert": true
    },
    "TeamSettings": {
        "SiteName": "Mattermost",
        "MaxUsersPerTeam": 50,
        "EnableTeamCreation": true,
        "EnableUserCreation": true,
        "RestrictCreationToDomains": ""
    },
    "SqlSettings": {
        "DriverName": "mysql",
        "DataSource": "mmuser:mmpass@tcp(localhost:3306)/mattermost?charset=utf8mb4,utf8",
        "DataSourceReplicas": [],
        "MaxIdleConns": 10,
        "MaxOpenConns": 10,
        "Trace": false,
        "AtRestEncryptKey": "7rAh6iwQCkV4cA1Gsg3fgGOXJAQ43QVg"
    },
    "LogSettings": {
        "EnableConsole": true,
        "ConsoleLevel": "DEBUG",
        "EnableFile": true,
        "FileLevel": "INFO",
        "FileFormat": "",
        "FileLocation": ""
    },
    "FileSettings": {
        "DriverName": "local",
        "Directory": "/opt/mattermost/data/",
        "EnablePublicLink": true,
        "PublicLinkSalt": "A705AklYF8MFDOfcwh3I488G8vtLlVip",
        "ThumbnailWidth": 120,
        "ThumbnailHeight": 100,
        "PreviewWidth": 1024,
        "PreviewHeight": 0,
        "ProfileWidth": 128,
        "ProfileHeight": 128,
        "InitialFont": "luximbi.ttf",
        "AmazonS3AccessKeyId": "",
        "AmazonS3SecretAccessKey": "",
        "AmazonS3Bucket": "",
        "AmazonS3Region": ""
    },
    "EmailSettings": {
        "EnableSignUpWithEmail": true,
        "SendEmailNotifications": true,
        "RequireEmailVerification": false,
        "FeedbackName": "",
        "FeedbackEmail": "",
        "SMTPUsername": "binaryhe@aliyun.com",
        "SMTPPassword": "284o8o352",
        "SMTPServer": "smtp.aliyun.com",
        "SMTPPort": "25",
        "ConnectionSecurity": "STARTTLS",
        "InviteSalt": "bjlSR4QqkXFBr7TP4oDzlfZmcNuH9YoS",
        "PasswordResetSalt": "vZ4DcKyVVRlKHHJpexcuXzojkE5PZ5eL",
        "ApplePushServer": "",
        "ApplePushCertPublic": "",
        "ApplePushCertPrivate": ""
    },
    "RateLimitSettings": {
        "EnableRateLimiter": true,
        "PerSec": 10,
        "MemoryStoreSize": 10000,
        "VaryByRemoteAddr": true,
        "VaryByHeader": ""
    },
    "PrivacySettings": {
        "ShowEmailAddress": true,
        "ShowFullName": true
    },
    "GitLabSettings": {
        "Enable": false,
        "Secret": "",
        "Id": "",
        "Scope": "",
        "AuthEndpoint": "",
        "TokenEndpoint": "",
        "UserApiEndpoint": ""
    }
    }
    ```
    - database
    
    ```
    CREATE DATABASE mattermost DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;
    GRANT ALL ON mattermost.* to 'mmuser'@'localhost' identified by 'mmpass';
    flush privileges;
    ```
- Initialize

```
cd bin
./platform
```
    - when run platform script, you will get `[CRIT] Failed to create index Error 1214: The used table type doesn't support FULLTEXT indexes` error, so you should change mysql db engine from innodb to MyISAM
    
    ```
    alter table Audits engine=myisam; 
    alter table ChannelMembers engine=myisam; 
    alter table Channels engine=myisam; 
    alter table IncomingWebhooks engine=myisam; 
    alter table OAuthAccessData engine=myisam; 
    alter table OAuthApps engine=myisam; 
    alter table OAuthAuthData engine=myisam; 
    alter table Posts engine=myisam; 
    alter table Sessions engine=myisam; 
    alter table Systems engine=myisam; 
    alter table Teams engine=myisam; 
    alter table Users engine=myisam;
    ```
    
    - run `platform` again and OK now. In order to run conviniently, I've write a straightforward start script for mattermost like below:
    
```
#!/bin/bash
# /etc/init.d/mattermost
# debian-compatible mattermost startup script.
#
### BEGIN INIT INFO
# Provides:          mattermost 
# Required-Start:    $remote_fs $syslog $network
# Required-Stop:     $remote_fs $syslog $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start mattermost at boot time
# Description:       Controls Mattermost Server
### END INIT INFO

# Mattermost Linux service controller script
home_dir=/usr/local/mattermost
cmd=$home_dir/bin/platform
pid=/var/run/mattermost.pid

case "$1" in
    start)
        start-stop-daemon --start --chdir $home_dir --background --name mattermost --make-pidfile --pidfile $pid --chuid mattermost --exec $cmd 
        ;;
    stop)
	start-stop-daemon --stop --pidfile $pid
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
        ;;
esac
```

DONE!
