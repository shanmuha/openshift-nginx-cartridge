## Openshift Nginx Cartridge

A cartridge for openshift that enables Nginx to be used as the web server.

This is a forked version from [here](https://github.com/gsterjov/openshift-nginx-cartridge). It has been updated to Nginx 1.7.9, some of the nginx modules are different, and the nginx config has been slightly changed.

### Installation

To install this cartridge use the cartridge reflector when creating an app

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

This seems to be an error in deployment, as the error.log is written to and shows errors correctly when viewing the file.

### Compile options

For those interested in upgrading this yourself at some point, perhaps for your own fork which you maintain, this is how I've done updates:

SSH to an OpenShift gear (rhc app-show appname to get the address), go to $OPENSHIFT_TMP_DIR ($ cd $OPENSHIFT_TMP_DIR) and do a wget for the latest nginx and pcre and unpack them. Navigate into the nginx folder and you can configure with a command like this:

	./configure --with-pcre=/tmp/pcre-8.36 --http-log-path=logs/access.log --error-log-path=logs/error.log --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module

copy the objs/nginx file to $OPENSHIFT_DATA_DIR (cp objs/nginx $OPENSHIFT_DATA_DIR) exit, and scp the file to your local machine to which you've cloned this repository, place the binary in the properly named usr folder in the repo, and update the manifest file in the metadata folder, and some of the files in the bin folder. 
