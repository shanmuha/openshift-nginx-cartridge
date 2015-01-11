## Scalable Openshift Nginx Cartridge

A cartridge for OpenShift that enables NGINX to be used. This OpenShift cartidge is scalable, I've tested it with 3 gears, it worked jolly good. No reason to believe it wouldn't work with a larger amount. 

This is a forked version of someone elses work from [here](https://github.com/gsterjov/openshift-nginx-cartridge). It has been updated to NGINX version 1.7.9, and some of the NGINX modules are different than the original. The NGINX config file supplied with the cartidge has also been changed.

### Installation

To install this cartridge you use the cartridge reflector from the command line. In the example below a "myapp" directory will be created in the location where the command is run as the git repo where your static web content will live will be cloned there.

	rhc create-app --scaling myapp http://cartreflect-claytondev.rhcloud.com/reflect?github=dustyherald/openshift-nginx-cartridge


### Configuration

The cartridge installs two config files. One at <code>$OPENSHIFT_NGINX_DIR/conf/nginx.conf</code> which gets loaded by the executable
and sets up specific app configuration such as logs and pid files.

The config then includes another nginx.conf which must exist at <code>$OPENSHIFT_REPO_DIR/nginx.conf</code>. This config should
contain all your server specific set up including which ip/port to listen on.

The repo nginx.conf is actually seen in your repository as <code>nginx.conf.erb</code> so environment variables can be used
in the config. Every time the server starts it first processes <code>nginx.conf.erb</code>.


A <code>public/</code> folder is included where static content is served by default. However, as can be seen in the <code>nginx.conf.erb</code> file it
is entirely configurable and only exists as a form of documentation.

### Known Issues

On pushing to OpenShift you'll see an error that looks something like this:

	remote: Starting Nginx
	remote: nginx: [alert] could not open error log file: open() "error.log" failed (2: No such file or directory)

This seems to be something going odd during the deployment process, as when NGINX is running the error.log is written to and shows errors correctly when viewing the file.

Unless you upgrade the amount of storage available you only have 1 gigabyte of space per gear, this includes the git repository which will track your static content folder. Due to the way this stuff is copied around the OpenShift gear, with 1 gig of space, you'll be able to fit somewhere around 300 megabytes of static content before youll need to add more storage to your gear(s).

### Compile options

For those interested in upgrading this yourself at some point, perhaps for your own fork which you maintain, this is how I've done updates:

SSH to an OpenShift gear (<code>rhc app-show appname</code> to get the address), go to $OPENSHIFT_TMP_DIR (<code>$ cd $OPENSHIFT_TMP_DIR</code>) and do a wget for the latest NGINX and PCRE and unpack them. Navigate into the created NGINX folder and you can configure the install, I've used this when compiling the binary included in this cartridge:

	./configure --with-pcre=/tmp/pcre-8.36 --http-log-path=logs/access.log --error-log-path=logs/error.log --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module

Run a <code>make</code> and when it is done copy the binary at <code>objs/nginx</code> to $OPENSHIFT_DATA_DIR (<code>cp objs/nginx $OPENSHIFT_DATA_DIR</code>) exit the SSH session. Now you can scp the file to your local machine (to which you've cloned this cartridge repository). Place the binary in the properly named <code>usr</code> folder, and update the manifest file in the metadata folder, and some of the files in the bin folder. Push it to a repo somewhere, test it on a new OpenShift gear.

