# basic-deploy

```
   ***************************************************
                      PLEASE NOTE
    This is a work in progress and subject to change.
   ***************************************************
```

---

## Purpose

A simple deployment script to pull code from github to your server and use symlinks to quickly switch from old code to new.

The script assumes that there is a repository with the same name as your vhost.


## Usage

Save the bash script to your server in `/usr/bin/` as `deploy`. To run it, ssh to your server and run `deploy`.

File system structure on server:

```
/var/www/vhosts/<vhost>/
                        current/
                                <timestamp>/
                        httpdocs -> current/<timestamp>/
```

The repository on github will have the same name as the vhost.


### Deploying site

```
deploy <vhost> <?commit>
```

Run the deploy command with the first parameter as the vhost. eg. `deploy alistairjcbrown.com`. An optional git commit id can be provided to deploy a particular commit. All deploys are from master.

Walkthrough of the deployment process:

1. Go to `/var/www/vhosts/<vhost>`

    - Fail if the directory does not exist

2. Check where `httpdocs` points to
   
   - Fail if `httpdocs` is not a symlink
   - Fail if `httpdocs` is not pointing to a directory within `current`

3. List all directories in `current`
    
    - Fail if list is not availble

4. Is the list of directories in `current` is less than the number of deploys to keep (configurable)?

   4.1. __Yes:__ Create directory called `<timestamp>` in `current`. Clone the repository from github into the new directory.
              
      - Fail if unable to create directory in `current`
      - Fail if unable to clone repository

   4.2. __No:__: Look for the last failed deploy (check directory for "-rolledback") or then the oldest deploy using the directory name timestamp. Rename the selected directory to `<timestamp>`.

      - Fail if unable to rename directory in current/

5. Checkout the latest master code (or a commit id if provided) from origin.
    - Fail if unable to checkout

6. Set file permissions (configurable)

7. Switch `httpdocs` symlink to point to the new deployed code.



### Rolling back deployment 

```
deploy <vhost> rollback <?timestamp>
```

Walkthrough of the rollback process:

1. Go to `/var/www/vhosts/<vhost>`

    - Fail if the directory does not exist

2. Check where `httpdocs` points to
   
   - Fail if `httpdocs` is not a symlink
   - Fail if `httpdocs` is not pointing to a directory within `current`

3. List all directories in `current`
    
    - Fail if list is not availble

4. Has the rollback provided a specific `<timestamp>` to roll back to?

   4.1. __Yes:__ Check that directory `<timestamp>` is in `current`.
              
      - Fail if directory `<timestamp>` is not in `current`

   4.2. __No:__: Use list of directories in `current` to get previous deploy using the directory name timestamp

      - Fail if no previous deployments.


5. Switch `httpdocs` symlink to point to the new deployed code.

6. Rename last deploy directory as bad by adding "-rolledback" to directory name,




## Configs

 * `previous_deploys` - Number of previous deploys to keep (defaults is 2)
 * `permissions` - Object containing information on the permissions to set for the new files.
   * `permissions.user` - User to set for the files, eg. `www-data`
   * `permissions.group` - Group to set for the files, eg. `www-data`
   * `permissions.file` - Permissions to set for the files, eg. `755`

### Assumptions

 * Git username has been set
   `git config --global user.name`
 * Repository will be cloned from Github
 * For automatic deploy, password has been saved or public key has been set
