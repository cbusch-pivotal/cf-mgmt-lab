# cf-mgmt-lab configuration

Document to show the flow of creating and using the Pivotal Service cf-mgmt tool.


1. Start by creating cf-mgmt client and secret in the PAS UAA

```
[~]$ uaac target uaa.system.busch.local
[~]$ uaac token client get admin -s <PAS UAA Admin Client password>
[~]$ uaac client add cf-mgmt \
  --name cf-mgmt \
  --secret pivotal1 \
  --authorized_grant_types client_credentials,refresh_token \
  --authorities cloud_controller.admin,scim.read,scim.write
```

2. Now create directory for your cf-mgmt assets and change into that directory

```
[~]$ mkdir cf-mgmt && cd cf-mgmt
```

3. Initialize the directory as a git repo

```
[~/cf-mgmt]$ git init
```

4. Extract the existing configuration from a foundation you have partially configured with orgs, space, and users. Good for extracting inital LDAP configuration, if it's being used

```
[~/cf-mgmt]$ cf-mgmt export-config --config-dir=config-me \
  --system-domain=system.busch.local --user-id=cf-mgmt --client-secret=pivotal1
```

5. Here are the files produced by the export-config

```
[~/cf-mgmt]$ tree .

cf-mgmt
└── config-me
    ├── asgs
    │   └── metrics-v1-5-api.json
    ├── busch
    │   ├── dev
    │   │   ├── security-group.json
    │   │   └── spaceConfig.yml
    │   ├── orgConfig.yml
    │   ├── prod
    │   │   ├── security-group.json
    │   │   └── spaceConfig.yml
    │   └── spaces.yml
    ├── cf-mgmt.yml
    ├── default_asgs
    │   └── default_security_group.json
    ├── ldap.yml
    ├── orgs.yml
    └── spaceDefaults.yml
```

6. From the cf-mgmt root directory, generate the Concourse pipeline

```
[~/cf-mgmt]$ cf-mgmt-config generate-concourse-pipeline

Generating pipeline....
1) Update vars.yml with the appropriate values
2) Using following command to set your pipeline in concourse after you have checked all files in to git
fly -t lite set-pipeline -p cf-mgmt -c pipeline.yml --load-vars-from=vars.yml
```

7. View the directory structure created, merged with the export-config.

```
[~/cf-mgmt]$ tree .

cf-mgmt
├── ci
│   └── tasks
│       ├── cf-mgmt.sh
│       └── cf-mgmt.yml
├── config-me
│   ├── asgs
│   │   └── metrics-v1-5-api.json
│   ├── busch
│   │   ├── dev
│   │   │   ├── security-group.json
│   │   │   └── spaceConfig.yml
│   │   ├── orgConfig.yml
│   │   ├── prod
│   │   │   ├── security-group.json
│   │   │   └── spaceConfig.yml
│   │   └── spaces.yml
│   ├── cf-mgmt.yml
│   ├── default_asgs
│   │   └── default_security_group.json
│   ├── ldap.yml
│   ├── orgs.yml
│   └── spaceDefaults.yml
├── pipeline.yml
└── vars.yml
```

8. Edit the vars.yml file

```
$ vi vars.yml
```

vars.yml file contents

```
# git repo uri
git_repo_uri: git@github.com:cbusch-pivotal/cf-mgmt-lab.git

# cf system domain
system_domain: system.busch.local

# user account with permission to create orgs/spaces and client secret for uaac for user_id
user_id: cf-mgmt
client_secret: pivotal1

# DEPRECATED - Use client_secret - password of user account with permission to create orgs/spaces
password:

# password to bind to ldap - only needed if using LDAP
ldap_password:

# logging level for cf-mgmt commands in the pipeline
log_level: INFO
```

9. Make git ignore the vars.yml file when committing. Don't want secrets shared.

```
[~/cf-mgmt]$ echo vars.yml >> .gitignore
```

10. Create a new git repo in github or gitlab and run commands to sync up your local work

```
[~/cf-mgmt]$ git add . --all
[~/cf-mgmt]$ git commit -m "initial commit"
[~/cf-mgmt]$ git remote add origin git@github.com:cbusch-pivotal/cf-mgmt-lab.git
[~/cf-mgmt]$ git push -u origin master
```

11. Set the pipeline, unpause, and run

```
[~/cf-mgmt]$ fly -t main set-pipeline -p cf-mgmt-pipeline -c ./pipeline.yml -l ./vars.yml
[~/cf-mgmt]$ fly -t main unpause-pipeline -p cf-mgmt-pipeline
[~/cf-mgmt]$ 
```
