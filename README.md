# Supported tags and respective `Dockerfile` links

- [`debian-jessie`, `debian`, `latest` (_Dockerfile_)](https://github.com/atmoz/sftp/blob/master/Dockerfile) [![](https://images.microbadger.com/badges/image/atmoz/sftp.svg)](http://microbadger.com/images/atmoz/sftp "Get your own image badge on microbadger.com")
- [`alpine-3.4`, `alpine` (_Dockerfile_)](https://github.com/atmoz/sftp/blob/alpine/Dockerfile) [![](https://images.microbadger.com/badges/image/atmoz/sftp:alpine.svg)](http://microbadger.com/images/atmoz/sftp "Get your own image badge on microbadger.com")

# Securely share your files with S3 filesystem baked-in with s3fs_fuse

Easy to use SFTP ([SSH File Transfer Protocol](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)) server with [OpenSSH](https://en.wikipedia.org/wiki/OpenSSH).
This is an automated build linked with the [debian](https://hub.docker.com/_/debian/) and [alpine](https://hub.docker.com/_/alpine/) repositories.

# Usage

- Define users as command arguments, STDIN or mounted in `/etc/sftp-users.conf`
  (syntax: `user:pass[:e][:uid[:gid[:dir1[,dir2]...]]]...`).
  - Set UID/GID manually for your users if you want them to make changes to
    your mounted volumes with permissions matching your host filesystem.
  - Add directory names at the end, if you want to create them and/or set user
    ownership. Perfect when you just want a fast way to upload something without
    mounting any directories, or you want to make sure a directory is owned by
    a user (chown -R).
- Mount volumes in user's home directory. Not supported with s3fs_fuse addition
  - The users are chrooted to their home directory, so you must mount the
    volumes in separate directories inside the user's home directory
    (/home/user/**mounted-directory**).
- s3fs is currently only supported with a single user-dir.
  Adding additional mounts for multiple users should be simple, but not in my original use-case.
  last-in wins currently

# Examples

## Simplest docker run example

```
docker run -e S3_BUCKET_NAME=yourbucket[:/OPTIONAL_SUBDIR] -e AWSACCESSKEYID=your_aws_access_key_id -e AWSSECRETACCESSKEY=your_aws_secret_key -e FTP_USER=user -e FTP_PASSWORD=12345 -e AWS_REGION=us-east-1 --security-opt apparmor:unconfined --cap-add mknod --cap-add sys_admin --device=/dev/fuse -p 2222:22 -d ghcr.io/alaabrahim/sftp-s3fs:master

```

User "user" with password "12345" can login with sftp and upload files to a folder in that container at /home/user/. Files uploaded this way are synced to S3 in the named S3_BUCKET_NAME.
The provided AWSACCESSKEYID must be associated with a role that has access permissions to the S3 bucket.
If you want to use a different region, you can set the AWS_REGION environment variable to the desired region. The default region if no region is supplied is ap-northeast-1.
If you are running the image on an EC2 instance with an IAM role that has access to the S3 bucket, you can omit the AWSACCESSKEYID and AWSSECRETACCESSKEY environment variables.

### Using Docker Compose:

Run the provided docker-compose.yml file, providing values in the environment for for AWSACCESSKEYID, AWSSECRETACCESSKEY, S3_BUCKET_NAME, USERNAME, and PASSWORD. eg:

```
AWSACCESSKEYID=your_aws_access_key_id AWSSECRETACCESSKEY=your_aws_secret_key S3_BUCKET_NAME=your_bucket_name FTP_USER=user FTP_PASSWORD=12345 docker-compose up -d
```

## Example Login: connect to a locally-running instance on the default docker IP:

```
sftp -P 2222 user@172.17.0.1
sftp> put somefile
sftp> ls
somefile
```

## Store users in config - NOTE: Multiple users not yet supported for s3fs (last-in wins for s3fs mount)

```
docker run \
    -v /host/users.conf:/etc/sftp-users.conf:ro \
    -v /host/share:/home/foo/share \
    -v /host/documents:/home/foo/documents \
    -v /host/http:/home/bar/http \
    -p 2222:22 -d ghcr.io/alaabrahim/sftp-s3fs:master
```

/host/users.conf:

```
foo:123:1001
bar:abc:1002
```
