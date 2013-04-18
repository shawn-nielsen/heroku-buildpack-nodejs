Heroku buildpack: Node.js
=========================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for Node.js apps.
It uses [NPM](http://npmjs.org/) and [SCons](http://www.scons.org/).

Usage
-----

Example usage:

    $ ls
    Procfile  package.json  web.js

    $ heroku create --stack cedar --buildpack http://github.com/heroku/heroku-buildpack-nodejs.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Node.js app detected
    -----> Vendoring node 0.4.7
    -----> Installing dependencies with npm 1.0.8
           express@2.1.0 ./node_modules/express
           ├── mime@1.2.2
           ├── qs@0.3.1
           └── connect@1.6.2
           Dependencies installed

The buildpack will detect your app as Node.js if it has the file `package.json` in the root.  It will use NPM to install your dependencies, and vendors a version of the Node.js runtime into your slug.  The `node_modules` directory will be cached between builds to allow for faster NPM install time.

Node.js and npm and Oracle Client versions
------------------------

You can specify the versions of Node.js and npm your application requires using `package.json`

    {
      "name": "myapp",
      "version": "0.0.1",
      "dependencies": {
      "db-oracle": "0.2.3",
        "generic-pool": "2.0.x"
     },
      "engines": {
        "node": ">=0.4.7 <0.7.0",
        "npm": ">=1.0.0",
        "oracle_client": "11.2.0"
      }
    }

To list the available versions of Node.js and npm, see these manifests:

http://heroku-buildpack-nodejs.s3.amazonaws.com/manifest.nodejs
http://heroku-buildpack-nodejs.s3.amazonaws.com/manifest.npm


Setup Oracle Package
--------------------

This gives CloudFoundry support for the 'db-oracle' npm package:
https://github.com/mariano/node-db-oracle

NodeJS versions are dependent on the above project version compatibility


 1.  Download Oracle C-Based Driver and SDK  to an empty directory from here:
  http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html
 
 2.  Create a cloudfoundry tar ball of the Oracle Client and SDK.  x.x.x is the client version (e.g. 11.2.0):
     
     unzip '*.zip';tar -zcvf oracle_client-x.x.x.tgz instantclient_*
     
 
 3. Upload the tgz zip file to the HTTP repo of your choice.  
    Update 'CUSTOM_ADDR' in bin/compile to point to that repo


 4. Create and upload a 'manifest.oracle_client' file that contains all allowable client versions
    e.g.  11.2.0 

   Upload to the same directory as your tgz file and would look simliar to:
   http://heroku-buildpack-nodejs.s3.amazonaws.com/manifest.npm  
   (but it would be manifest.oracle_client)
 
 5. Make sure your package.json file contains the 'oracle_client' version
 
 7. Deploy application using:
   cf push --buildpack=https://<gitRepo>/<forkedRepo>/heroku-buildpack-nodejs 

 8. When you deploy select at least 128MB RAM


Hacking
-------

To use this buildpack, fork it on Github.  Push up changes to your fork, then create a test app with `--buildpack <your-github-url>` and push to it.

To change the vendored binaries for Node.js, NPM, and SCons, use the helper scripts in the `support/` subdirectory.  You'll need an S3-enabled AWS account and a bucket to store your binaries in.

For example, you can change the default version of Node.js to v0.6.7.

First you'll need to build a Heroku-compatible version of Node.js:

    $ export AWS_ID=xxx AWS_SECRET=yyy S3_BUCKET=zzz
    $ s3 create $S3_BUCKET
    $ support/package_nodejs 0.6.7

Open `bin/compile` in your editor, and change the following lines:

    DEFAULT_NODE_VERSION="0.6.7"
    S3_BUCKET=zzz
    ORACLE_ADDR="http://yourRepoThatContainsPackagedClient"

Commit and push the changes to your buildpack to your Github fork, then push your sample app to Heroku to test.  You should see:

    -----> Vendoring node 0.6.7
