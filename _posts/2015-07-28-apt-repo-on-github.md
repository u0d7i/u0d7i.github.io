---
layout: post
title: "APT repo on github"
date: 2015-07-28 08:30
---

Case: repack/merge several .deb packages into a new one, and distribute via APT repo, hosted on github pages.

Extract existing debian package:

~~~
$ wget -c http://repository.maemo.org/extras-devel/pool/fremantle/free/c/cryptsetup/cryptsetup_1.0.7-12maemo0_armel.deb
$ dpkg-deb -x cryptsetup_1.0.7-12maemo0_armel.deb cryptsetup
$ dpkg-deb -e cryptsetup_1.0.7-12maemo0_armel.deb cryptsetup/DEBIAN
~~~

After modifications (deoptifying in this case) and merging, editing control files in DEBIAN/ subdir
(see [Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-controlfields.html) for details on
Control files and their fields), build it to a new .deb:

~~~
$ dpkg-deb -b cryptsetup/ ./
~~~

In this case new package is created in current dir, named by fields in DEBIAN/control file:

~~~
dpkg-deb: building package 'cryptsetup-deopt' in './/cryptsetup-deopt_1.0.7-maemo0_armel.deb'.
~~~

You can define package filename instead of ./ or omit it (package is named by the dir then).

Create github repo for an APT repo (from the cmdline, using API, of course).
Export github username and pass as variables. I do it by sourcing a file, containing

~~~
export GHU="<github_user>"
export GHP="<gitgub_password>"
~~~

~~~
$ source ./github_credentials.txt
~~~

Then we can use github API via curl to create github repo:

~~~
$ curl -u "${GHU}:${GHP}" https://api.github.com/user/repos -d '{"name": "apt-repo", "auto_init": true}'
~~~

where "apt-repo" stands for repository name.
Make a fresh clone of newly created repo:

~~~
$ git clone git@github.com:${GHU}/apt-repo
~~~

Create a gh-pages branch, making it orphan, without any parents, and clean out its content:

~~~
$ cd apt-repo/
$ git checkout --orphan gh-pages
$ git rm -rf .
~~~

Add some dummy content and push:

~~~
$ echo "APT repo"  > index.html
$ git add index.html
$ git commit -a -m "Initial commit"
$ git push origin gh-pages
~~~

Install reprepro:

~~~
$ sudo apt-get install reprepro
~~~

Generate signing key for the repo:

~~~
$ gpg --gen-key
~~~

Configure reprepro as described [here](https://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro), and init APT repo,
by adding previously repacked debfile to it:

~~~
$ mkdir conf
$ vi conf/distributions
$ cat <<EOF >> conf/options
verbose
basedir .
ask-passphrase
EOF
$ reprepro includedeb fremantle ../cryptsetup-nonopt_1.0.7maemo0_armel.deb
~~~

Export public key:

~~~
$ gpg --armor --output pubkey.gpg --export <key-id>
~~~

A final touch by adding some useful info about the repo to index.html (gitpages do not use directory listings
and you must have index files, or you get 404 on directory).

~~~
$ vi index.html
~~~

Commit, push and enjoy:

~~~
$ git add --all
$ git commit -m "APT repo init"
$ git push origin gh-pages
~~~

Result is here: [http://u0d7i.github.io/apt-repo/](http://u0d7i.github.io/apt-repo/)

