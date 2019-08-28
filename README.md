Free Law (Virtual) Machine v2.1.0
==================================

## NOW OBSOLETE

This repo is now obsolete, replaced by docker. See details in the CourtListener repository.

--------------------

Want to help develop Free Law Project functionality?

Follow the steps below to build a development environment for the [Free Law Project](https://github.com/freelawproject) components like the [CourtListener](https://github.com/freelawproject/courtlistener) website.  You will also be able to build new or custom Vagrant boxes and contribute back to this freelawmachine repository.

This repository and the steps supercede the manual process [described](https://github.com/freelawproject/courtlistener/wiki/Installing-CourtListener-on-Ubuntu-Linux) in the CourtListener [wiki](https://github.com/freelawproject/courtlistener/wiki).  They are intended to make the creation of a dev environment about as easy as a `vagrant up` command.

## Step 1:  Install prerequisites

Install the following.  Use a high-speed connection.  (These components are large, so avoid connections for which you will have to pay $$ for the sizes of your downloads!)

* [Vagrant 2.1.2 or greater](https://www.vagrantup.com)
* [Virtualbox 5.1.4 or greater](https://www.virtualbox.org/)
* (Optional) [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) OR [GitHub Desktop](https://desktop.github.com)
* (Optional) [Packer 0.10.1](https://packer.io/downloads.html) (if you want to contribute to this freelawmachine repository)

## Step 2:  Install CourtListener

1. Make sure you have the base requirements of *Vagrant* and *VirtualBox* installed and up to date.

2. Either download a copy of the [Vagrantfile](Vagrantfile) file into a local directory or, if you want to contribute back to this freelawmachine repository, fork it within GitHub to your GitHub account, create a branch, and use git or GitHub Desktop to clone that branch to a local directory.

3. Open a terminal or command line and change to that directory (which now contains the 'Vagrantfile' file).

  `cd <directory>`

4. This is where the ✨magic✨ happens! Just run:

  `vagrant up`

   And the base box will be pulled down from its hosted location, installed into VirtualBox, started, and last-mile provisioning steps will take place.  _This now includes cloning the latest copy of CourtListener for you!_

5. Now to log into the box...it's a simple:

  `vagrant ssh`

  For the password, enter 'vagrant'.

6. If you haven't used the machine yet, you'll need to do some basic CourtListener provisioning steps that currently aren't handled (yet) by our Vagrant provisioning scripts.  Run the following commands in the terminal:

  ``` bash
  cd /var/www/courtlistener
  pip install -U -r requirements.txt
  ./manage.py migrate
  ./manage.py syncdb #to create the admin user
  ./manage.py changepassword admin  #to set the password for the Django admin user
  ```

7. Other commands that may be useful within the `/var/www/courtlistener` directory:

``` bash
./manage.py shell  #Opens a python shell with django stuff nicely imported.
./manage.py dbshell  #Opens a postgres shell, handling password stuff for you.
```

##  Step 3:  Start CourtListener

From within the `/var/www/courtlistener` directory, simply use the [Django](https://www.djangoproject.com/)'s manage scripts as follows to launch the app. Make sure you pay close attention to adding the IP address and port number so it plays nice with Vagrant's NAT networking and the box's network adapters.

```bash
./manage.py runserver 0.0.0.0:8000
```

_For more details on why this is, check out this StackOverflow
[post](http://stackoverflow.com/questions/1621457/about-ip-0-0-0-0-in-django)._

Fire up your browser (on your local machine!), navigate to: [http://127.0.0.1:8000](http://127.0.0.1:8000), and confirm you've got a local instance that looks like [courtlistener.com](https://www.courtlistener.com/).

You can access the administrative interface at [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) and with the username "admin" and the password you set above.

## Step 4:  Scrape some court opinions!

You can easily load some opinions into your CourtListener instance by scraping some courts using the [Juriscraper](https://github.com/freelawproject/juriscraper/) commands built into CourtListener. In either a new SSH session/shell or after cancelling (`ctrl-c`) the "runserver" command, try:

```bash
./manage.py cl_scrape_opinions \
  --courts juriscraper.opinions.united_states.federal_appellate.ca1 \
  --rate 5
```

CourtListener will spin up a Juriscraper instance for the given court and load the result items into the PostgreSQL instance as well as feed the items to the Solr instance. (If you don't get any items with "ca1", then try another like "ca6".)

Once complete after a successful crawl or a timeout or after you manually kill the Juriscraper instance with some `ctrl-c`'s, then you need to tell Solr to commit changes and make the new docs in the index go live:

```sh
./manage.py cl_update_index --do-commit --type opinions \
  --solr-url http://127.0.0.1:8983/solr/collection1
```

You should now have some results on the landing page as well as fully searchable
opinions!

You can inspect the Solr index cores directly using your browser:
[http://localhost:8983/solr/#/](http://localhost:8983/solr/#/)

NOTE:  If you encounter solr errors when adding data, see freelawmachine issue #[28](https://github.com/freelawproject/freelawmachine/issues/28).

## Modifying your local CourtListener website and contributing back to its repository

You should now have a working local instance of the CourtListener website.  You should also have a copy of the CourtListener repository as a sub-folder.  To modify it and contribute your changes back to its repository, follow the [GitHub Flow](https://guides.github.com/introduction/flow/).

## Working with and contributing to the FreeLawMachine repository

### Building a new Vagrant box

Here's how to crank out a box if you've got the Requirements above. Depending on your network connection, CPU, disk, etc. this could take anywhere from 20 mins to maybe 30 mins. Be patient :-)

All of the tools required to build the box using [Packer](https://packer.io) are contained in the [packer](./packer) directory of this project. [Ansible](https://github.com/ansible/ansible) is installed into the VM image itself, so there's no need to have it installed on your local machine.

  1. Grab the latest Free Law Machine source:

    git clone https://github.com/freelawproject/freelawmachine

  2. Jump into the packer directory:

    cd packer

  3. Build the box! (Yes, it's that simple. Since it's configured headless, it
  may appear nothing is happening for a little while. It's ok. This could take
  about 20-30 minutes!)

    packer build freelawbox64.json

  4. Install the Vagrant box on your local machine. The new _.box_ file will
  have a timestamp in the filename, so make sure to add the correct file:

    vagrant box add freelawbox64-{version}.box --name freelawproject/freelawbox64

Voila! You now have a new Vagrant box installed locally. You can even share the
_.box_ file with others the old fashioned way, host it at a URL, etc. (Vagrant
supports pulling boxes via URL.)

### Building your own Vagrant box using the Ansible playbooks

If you aren't looking to build a Vagrant base box and instead just want to take
a vanilla Ubuntu 14.04 base image (e.g. something like _ubuntu/trusty64_ or
_/boxcutter/ubuntu1404_), you can install
[Ansible](https://github.com/ansible/ansible) locally and use the same
playbooks used when building from scratch with Packer.

  0. Install the latest Ansible via either `pip install ansible` or other means.

  1. Change into the `ansible` directory where there's already a stubbed-out
  Vagrantfile waiting for you.

  2. The playbooks are executable, so just run: `./freelawmachine.yml` from your
  command line.

  3. Get a drink because you could be waiting about 20 minutes or so :-)

### Vagrant Tips
If you're playing around, here are some things to remember:
* Vagrant installs boxes typically in your home directory under something like
`.vagrant.d`. Make sure you use the same box name when using `vagrant box add`
and it will replace that existing box. (You probably need to add `--force` or
  something to the command, btw.)
* Vagrant keeps instances of vm's in the local directory where your Vagrantfile
is and where you are running `vagrant up`. Do a simple `vagrant destroy` to
wipe it out when you've built a new box version and want to start over.

Also see [a packer template for Vagrant from Hashicorp](https://github.com/hashicorp/atlas-packer-vagrant-tutorial.git).

## FreeLawMachine Change Log

### Changes in 2.1.0
Fixed issues related to Solr core creation/destruction during Django tests
Added new box version! The new freelawproject/freelawbox64-desktop base box provides an X11/XFCE4 desktop environment with Chromium installed and ready for Selenium tests

### Changes in 2.0.2
Fix for missing /reloadCache Solr endpoint

### New in Version 2.0.0!
Major changes for users of 1.6:

A wild Ansible appears! Provisioning is now done via Ansible playbooks instead of nasty hard to maintain shell scripts.
The box is set up closer to the original wiki specs, so it uses a Python Virtual Environment, but it should be auto-activated for you upon ssh login.
Similar to v1.6, you only need the Vagrantfile to run the box (assuming you have virtualbox and vagrant). It will do the rest.
Deprecation Warning! We're removed support for 32-bit boxes. New standard is to use 64-bit to mimic exact same packages as used in Production.

### Major changes for users of 1.5 and earlier:

The flp directory is no longer used or needed. If you want to pre-clone a copy of CourtListener, simply clone it to the directory where the Vagrantfile resides. This is not required as the box should do that for you at startup.
VirtualBox is now told to allocate 2 GB of memory to the box. This should be enough for dev use of CourtListener and small enough not to impact most hosts.
