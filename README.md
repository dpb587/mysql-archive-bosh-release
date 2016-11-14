# mysql-archive-bosh-release

To create a backup from a MySQL database and restore it later.


## Usage

### Backup

To prepare for a backup, configure the [`mysql-backup`](jobs/mysql-backup/spec) job with `mysql` and `storage` links, as well as the `database` and `storage_bucket` properties...

    jobs:
      - name: "mysql-backup"
        release: "mysql-archive"
        consumes:
          mysql: { from: "mysql", deployment: "mysql" }
          storage: { from: "storage", deployment: "storage" }
        properties:
          database: "wordpress"
          storage_bucket: "my-wordpress-backups"

Configure the instance as an `errand` or `service`. As an `errand`, invoke it like normal...

    $ bosh run-errand backup-database

As a `service`, invoke it manually (it will not run on its own)...

    $ bosh ssh -c /var/vcap/jobs/mysql-backup/bin/run database/0

Or colocate with and configure the [`mysql-backup-scheduler`](jobs/mysql-backup-scheduler) with a `schedule` property...

    jobs:
      - name: "mysql-backup"
        ...snip...
      - name: "mysql-backup-scheduler"
        release: "mysql-archive"
        properties:
          schedule: "0 1 * * *"

Once completed, a compressed file might be available at...

    https://backup.storage.internal/my-wordpress-backups/backup-20161101T010004Z.sql.xz.gpg


### Restore

To prepare for a restore, configure the [`mysql-restore`](jobs/mysql-restore/spec) job with `mysql` and `storage` links, as well as the `database` and `storage_bucket` property...

    jobs:
      - name: "mysql-restore"
        release: "mysql-archive"
        consumes:
          mysql: { from: "mysql", deployment: "mysql" }
          storage: { from: "storage", deployment: "storage" }
        properties:
          database: "wordpress"
          storage_bucket: "my-wordpress-backups"


Configure the instance as an `errand` and run it like normal...

    $ bosh run-errand restore-database


### Encryption

Use the `public_key` property of `mysql-backup` to encrypt backups with `gpg`, and use the `private_key` property of `mysql-restore` to decrypt backups.


## License

[MIT License](./LICENSE)
