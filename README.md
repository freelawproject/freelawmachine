Free Law (Virtual) Machine v1.3.0
==================================

This project is designed to provide automation around building a ready-to-run
development environment for the [Free Law Project](https://github.com/freelawproject) using Vagrant boxes.

Hopefully, it will do away with the manual process described
[here](https://github.com/freelawproject/courtlistener/wiki/Installing-CourtLitener-on-Ubuntu-Linux)
and make having a dev environment as easy as `vagrant up`.

## Requirements
* [Vagrant 1.7.4](https://www.vagrantup.com)
* [Virtualbox 5.x](https://www.virtualbox.org/)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* (Optional) [Packer 0.8.6](https://packer.io/downloads.html)


And a high-speed network connection since the VM will need to pull down
packages! (You are installing Ubuntu...so do this via a cellular data
connection at your own $$ peril!)

## How to Get Started

There are two phases to using this project (for the time being):

1. *Initializing the Vagrant Box* - this is the most common Vagrant task and
is what brings your local Free Law development environment online and ready
for use in development or testing CourtListener.

2. *Building the Vagrant Box (Optional)* - if you want to customize how the box
is built or you want to contribute to this project.


### Creating a local Free Law Development Environment

You have two choices: build the box from scratch (the hard way) or grab a hosted copy (the easy way).

#### The Easy Way

1. In the root of this project, change to the `vagrant` dir:

  `cd vagrant`

2. We need to now grab the CourtListener and Juriscraper source. At time of
this writing, the following will grab the two relevant development branches:

  `git clone -b opinion-split https://github.com/freelawproject/courtlistener.git`

  `git clone -b issue-59-pip https://github.com/freelawproject/juriscraper`

3. Return to the root of this project.

  `cd ..`

4. Now this is where the ✨magic ✨ happens! (The hosted box that is prebuilt
  will be pulled down and installed by Vagrant.) Just run:

  `vagrant up`

5. Now to log into the box, it's a simple:

  `vagrant ssh`

6. If you haven't used the machine yet, you'll need to do some basic
CourtListener provisioning steps that currently aren't handled (yet) by our
Vagrant provisioning scripts.

  ``` bash
  cd /var/www/courtlistener
  ./manage.py migrate
  ```

#### The Hard Way
Don't like the easy way? Skip ahead to "Building the Vagrant Box (Optional)"
and complete those steps first, then run through the "Easy Way" steps. This is
not for the feint of heart.


#### Starting Solr
If Solr is not up and running, you will have to start it manually.
  
  `sudo service solr start`



#### Starting CourtListener
From within the `/var/www/courtlistener` directory, simply use Django's manage
scripts to launch the app. Make sure you pay close attention to adding the IP
address and port number so it plays nice with Vagrant's NAT networking and the
box's network adapters.

```bash
./manage.py runserver 0.0.0.0:8000
```

For more details on why this is, check out this StackOverflow
[post](http://stackoverflow.com/questions/1621457/about-ip-0-0-0-0-in-django).

Fire up your browser (on your local machine!) and confirm you've got a local
instance that looks like [courtlistener.org](https://www.courtlistener.com/).

  Navigate to: [http://localhost:8888](http://localhost:8888)

You'll see there's no content/opinions, but you can load them using the
[Juriscraper](https://github.com/freelawproject/juriscraper/) commands built
into CourtListener. In either a new SSH session/shell or after cancelling
(`ctrl-c`) the "runserver" command, try:

```bash
./manage.py cl_scrape_opinions \
  --courts juriscraper.opinions.united_states.federal_appellate.ca1 \
  --rate 5
```

CourtListener will spin up a Juriscraper instance for the given court scraper
and load the output into the PostgreSQL instance as well as feed the results to
the Solr instance. Once complete (after a timeout) or after you manually kill it
with some `ctrl-c`'s, you need to tell Solr to commit changes and make the new
docs in the index go live:

```bash
./manage.py cl_update_index --do-commit --type opinions \
  --solr-url http://127.0.0.1:8983/solr/collection1
```

Then (if you don't have it running in another shell), bring CourtListener back
up with:

```bash
./manage.py runserver 0.0.0.0:8000
```

You should now have some results on the landing page as well as fully searchable
opinions!


To check for logs during the setup, Open your browser and
navigate to : [http://localhost:8999](http://localhost:8999)


### Building the Vagrant Box (Optional)
Here's how to crank out a box if you've got the Requirements above. Depending
on your network connection, CPU, disk, etc. this could take anywhere from 5
mins to maybe 30 mins. Be patient :-)

All of the tools required to build the box using [Packer](https://packer.io)
are contained in the [packer](./packer) directory of this project.

  0. Grab the latest Free Law Machine source:

    `git clone https://github.com/freelawproject/freelawmachine`

  1. Jump into the packer directory:

    `cd packer`

  2. Build the box! (Yes, it's that simple. Since it's configured headless, it
  may appear nothing is happening for a little while. It's ok.)

    `packer build flm-packer.json`

  3. Install the Vagrant box on your local machine. The new _.box_ file will
  have a timestamp in the filename, so make sure to add the correct file:

    `vagrant box add freelawbox-{timestamp}.box --name freelaw/freelawbox32`

Voila! You now have a new Vagrant box installed locally. You can even share the
_.box_ file with others the old fashioned way, host it at a URL, etc. Vagrant
supports pulling boxes via URL, so that's an option! You can always host your
own version in Hashicorp's [Atlas](https://atlas.hashicorp.com/) so others
can find and use it!


## Development Tips
If you're playing around, here are some things to remember:
* Vagrant installs boxes typically in your home directory under something like
`.vagrant.d`. Make sure you use the same box name when using `vagrant box add`
and it will replace that existing box. (You probably need to add `--force` or
  something to the command, btw.)
* Vagrant keeps instances of vm's in the local directory where your Vagrantfile
is and where you are running `vagrant up`. Do a simple `vagrant destroy` to
wipe it out when you've built a new box version and want to start over.


## Various Details
* Current box is based on Ubuntu Server 32-bit (for now) 14.04 LTS Trusty Tahr
* Trying to target dev's machines that have:
  * 8 GB RAM
  * [Virtualbox 5.x](https://www.virtualbox.org/)
  * Iffy VT-X support or running inside another VM (hence 32-bit VM)

## Possible Future Goals
* VMWare support (cost $$$?)
* 64-bit support - does this matter?
* Help stub out some complicated dependencies (like Solr) to cut down on box
  size and complexity.

## Some References
[A packer template for Vagrant from Hashicorp](https://github.com/hashicorp/atlas-packer-vagrant-tutorial.git)
