---
layout: post
title:  "Switching AWS profiles in Terminal faster"
date:   2015-10-29 21:27:08
categories: tech
---

I am using several AWS accounts and previously I used to edit `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` variables by uncommenting pairs of lines in includable file.

Today I found [several] [code snippets] and fixed them to work for me.

Install:
--------
My `~/.aws/credentials` (`chmod 600 ~/.aws/credentials`) look like this:

    [project1-anton]
    aws_access_key_id = AKIAIODSECRET7SIQF1Q
    aws_secret_access_key = 3jSecRetGoeSHerebhyLr

    [project1-anton-admin]
    aws_access_key_id = AKIAIODSECRET7SIQF2Z
    aws_secret_access_key = 12ASecRetGoeSHerebhyLr

Part of includable shell script (`~/.aws_aliases`, `chmod 755 ~/.aws_aliases`):

    function _awsListAll() {
 
    credentialFileLocation=$(env | grep AWS_SHARED_CREDENTIALS_FILE | cut -d= -f2);
    if [ -z $credentialFileLocation ]; then
        credentialFileLocation=~/.aws/credentials
    fi
 
    while read line; do
        if [[ $line == "["* ]]; then echo "$line"; fi;
    done < $credentialFileLocation;
    };

    function _awsSwitchProfile() {
    if [ -z $1 ]; then  echo "Usage: awsp profilename"; return; fi
    
    exists="$(aws configure get aws_access_key_id --profile $1)"
    if [[ -n $exists ]]; then
       export AWS_DEFAULT_PROFILE=$1;
       export AWS_ACCESS_KEY_ID="$(aws configure get aws_access_key_id --profile $1)"
       export AWS_SECRET_ACCESS_KEY="$(aws configure get aws_secret_access_key --profile $1)"
       echo "Switched to AWS Profile: $1";
       aws configure list
    fi
    };

    complete -W "$(cat $HOME/.aws/credentials | grep -Eo '\[.*\]' | tr -d '[]')" _awsSwitchProfile

Relevant part of `~/.bash_profile`:
```
source ~/.aws_aliases

alias awsall="_awsListAll"

alias awsp="_awsSwitchProfile"

alias awswho="aws configure list"
```

Usage:
-----
List all AWS profiles:
```
$ awsall
```

Show your current AWS profile:
```
$ awswho
```

Switch AWS profile:
```
$ awsp
```
```
$ awsp project1-anton
```

Now there is autocomplete for available profiles in terminal and I should only edit keys in single file.

[several]:                        https://github.com/antonosmond/bash_profile/blob/master/.bash_profile
[code snippets]:                  http://www.jayway.com/2015/09/25/aws-cli-profile-management-made-easy/
