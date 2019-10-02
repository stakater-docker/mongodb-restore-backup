# mongodb-restore-backup
A repo containing image to backup &amp; restore mongodb snapshots

## Usage
### Running as a single container
```
docker run -d \
  --env AWS_ACCESS_KEY_ID=awsaccesskeyid \
  --env AWS_SECRET_ACCESS_KEY=awssecretaccesskey \
  --env BUCKET=s3bucket
  --env BACKUP_FOLDER=a/sub/folder/path/ \
  --env INIT_BACKUP=true \
  --env MONGODB_HOST=mongodb.host \
  --env MONGODB_PORT=27017 \
  --env MONGODB_USER=admin \
  --env MONGODB_PASS=password \
  stakater/mongo-backup-restore-s3:0.0.4
```

If you link `stakater/mongo-backup-restore-s3` to a mongodb container with an alias named mongodb, this image will try to auto load the `host`, `port`, `user`, `pass` if possible. Like this:

```
docker run -d \
  --env AWS_ACCESS_KEY_ID=myaccesskeyid \
  --env AWS_SECRET_ACCESS_KEY=mysecretaccesskey \
  --env BUCKET=mybucketname \
  --env BACKUP_FOLDER=a/sub/folder/path/ \
  --env INIT_BACKUP=true \
  --link my_mongo_db:mongodb \
  stakater/mongo-backup-restore-s3:0.0.4
```
### Running in K8s

`mongo-restore.yaml` contains a manifest for `StatefulSet` which includes a pod running two containers. The first container is the actual mongodb server that is used by the application and the second is the backup sidecar container. This manifest can be used for any deployment of mongo db instance which includes the automatic backup/restore capabilities. By default AWS credentials and bucket name is fetched from the secret named `mongo`

To use it in k8s configure the variables (in Secrets or manifest) and use the following command to deploy:

`kubectl apply -f mongo-restore.yaml -n <namespace>` 

### ENVIRONMENT variables

`AWS_ACCESS_KEY_ID` - your aws access key id (for your s3 bucket)

`AWS_SECRET_ACCESS_KEY`: - your aws secret access key (for your s3 bucket)

`BUCKET`: - your s3 bucket

`BACKUP_FOLDER`: - name of folder or path to put backups (eg `myapp/db_backups/`). defaults to root of bucket.

`MONGODB_HOST` - the host/ip of your mongodb database

`MONGODB_PORT` - the port number of your mongodb database

`MONGODB_USER` - the username of your mongodb database. If MONGODB_USER is empty while MONGODB_PASS is not, the image will use admin as the default username

`MONGODB_PASS` - the password of your mongodb database

`MONGODB_DB` - the database name to dump. If not specified, it will dump all the databases

`EXTRA_OPTS` - any extra options to pass to mongodump command

`CRON_TIME` - the interval of cron job to run mongodump. `0 1 * * *` by default, which is every hour at 0 mins like 1:00, 2:00...

`TZ` - timezone. default: `US/Eastern`

`CRON_TZ` - cron timezone. default: `US/Eastern`

`INIT_BACKUP` - if defined, create a backup when the container launched

`INIT_RESTORE` - if defined, restore from latest when container is launched

`DISABLE_CRON` - if defined, it will skip setting up automated backups. good for when you want to use this container to seed a dev environment.


## Acknowledgements

* forked from [halvves](https://github.com/halvves/mongodb-backup-s3)

## Note

This repository is not created through terraform
