The below is run as root. It should work without root but the ownership of the files may be wrong
unless a transform is added to mcollective.mog.

Build dependencies:

    sudo pkg install ruby-18 system/header developer/gcc-3

Set some variables and get the source.

    PKGDATA=/var/tmp/mco_package
    APP_VERSION=2.7.0
    PKG_VERSION=0.1

    mkdir $PKGDATA

    mkdir -p $PKGDATA/var/ruby/1.8/gem_home
    gem18 install --install-dir $PKGDATA/var/ruby/1.8/gem_home --no-ri --no-rdoc stomp json

    git clone https://github.com/puppetlabs/marionette-collective.git
    (cd marionette-collective; git checkout $APP_VERSION || exit)

    # Set the version that the app reports.
    gsed -ie "s/@DEVELOPMENT_VERSION@/$APP_VERSION/" marionette-collective/lib/mcollective.rb

Edit marionette-collective/ext/solaris11/Makefile:

 * Set DESTDIR=/var/tmp/mco_package
 * Change "cp mc-*" to "-cp mc-*" (i.e. add a dash so it ignores error)

Make files into the temporary directory, and run the various `pkg*` commands to produce the final `mcollective.p5m.3.res` manifest.

    (cd marionette-collective; make -f ext/solaris11/Makefile install)

    rm $PKGDATA/COPYING
    mkdir -p $PKGDATA/lib/svc/manifest/application

    cp mcollective.xml $PKGDATA/lib/svc/manifest/application/mcollective.xml

    cp mcollective-start $PKGDATA/usr/sbin/

    pkgsend generate $PKGDATA | pkgfmt > mcollective.p5m.1

    pkgmogrify -DAPP_VERSION=$APP_VERSION -DPACKAGE_VERSION=$PKG_VERSION -DARCH=`uname -p` mcollective.p5m.1 mcollective.mog | pkgfmt > mcollective.p5m.2

    pkgdepend generate -md $PKGDATA mcollective.p5m.2 | pkgfmt > mcollective.p5m.3

    pkgdepend resolve -m mcollective.p5m.3

Publish to local repo (or wherever):

    pkgsend publish -s http://localhost:9001 -d $PKGDATA mcollective.p5m.3.res
