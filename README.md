Install cf-mysql service into TryCF
===================================

This project contains instructions and helper scripts/templates for deploying the cf-mysql service broker to your [TryCF](https://trycf.starkandwayne.com) Cloud Foundry environment.

TryCF is built with bosh-lite, and as such we can talk to it like any other BOSH. We can upload releases, stemcells and deploy them. In fact, any BOSH release that includes bosh-lite or warden spiff templates, or `make_manifest` scripts, or example manifests is primed for you to experiment with on your TryCF VM.

Requirements
------------

- Install the Cloud Foundry CLI https://github.com/cloudfoundry/cli/releases

Steps
-----

Target the internal BOSH within your TryCF VM.

```
bosh target <TRYCF_IP>
```

Run the following steps from within this project:

```
bosh upload release https://community-shared-boshreleases.s3.amazonaws.com/boshrelease-cf-mysql-8.tgz
./bosh-lite/make_manifest_spiff_mysql
bosh -n deploy
bosh run errand broker-registrar
```

Usage
-----

```
$ cf m
service   plans       description
p-mysql   100mb-dev   A MySQL service for application development and testing

$ cf cs p-mysql 100mb-dev my-sql
Creating service my-sql in org default / space development as admin...
OK

$ cf s
name     service   plan        bound apps
my-sql   p-mysql   100mb-dev
```

The above commands are functional abbreviations for the following commands:

- `cf marketplace`
- `cf create-service`
- `cf services`

Debugging
---------

To access the internal jobs, you need to SSH into the TryCF VM on AWS.

First, get the public IP (`107.20.174.183` in examples below):

```
$ bosh target
Current target is https://107.20.174.183:25555 (Bosh Lite Director)
```

Then download the pem key from your TryCF introduction email.

```
$ cp ~/Downloads/trycf.pem ~/.ssh/trycf.pem
$ chmod 600 ~/.ssh/trycf.pem
```

Now you can ssh:

```
$ ssh ubuntu@107.20.174.183 -i ~/.ssh/trycf.pem
```

Upgrade to new cf-mysql-release final releases
----------------------------------------------

For future maintainers of this project, when cf-mysql-release cuts future versions:

- create a tarball of the final release (within `cf-mysql-release` project: `bosh create release --with-tarball releases/cf-mysql-9.yml`)
- upload tarball to public blobstore (within `community-bosh-releases` project: `bosh share release ../cf-mysql-release/releases/cf-mysql-9.tgz`)
- document release tarball in `community-bosh-releases` project README
- update this project's README
- copy over cf-mysql templates (within `cf-mysql-release` project: `git checkout v9; cp -R templates ../trycf-mysql/`)

Now test the instructions above.
