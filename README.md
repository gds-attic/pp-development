# pp-development

Performance Platform development environment

## Basic setup

- Check / install dependencies with `GOVUK_DEPS=true ./install.sh` Warnings
  about the ``pp-deployment`` repository can be safely ignored.
- Start the virtual machine with `vagrant up` (this will provision the machine)
-- VMWare users may [hit an error](http://superuser.com/questions/511679/getting-an-error-trying-to-set-up-shared-folders-on-an-ubuntu-instance-of-vmware)
- SSH on to the machine with `vagrant ssh`
- Install the "bowl" command by doing $ sudo gem install bowler
- Install dependencies for each required app in /var/apps by following the
  instructions in their README files.

## Running apps

- SSH on to the machine with `vagrant ssh`
- Change to the development directory with `cd /var/apps/pp-development`
- Start the apps individually with ie `bowl backdrop_read` or all together with
  `bowl performance`
- See the file ``Pinfile`` which is used by bowl.

## Routing from your local machine to your VM

- Each backdrop app runs on its own local port ie 3038, 3039.
- From inside the VM, you can access apps directly at ie localhost:3038
- From outside the VM, requests to port 80 are routed to the correct app based
  on domain name, HTTP method and URL.
- Note the IP of your VM as defined in ``Vagrantfile``. It's worth confirming
  by running ``ifconfig`` inside the VM.
- Add a hostname entry to your local machine's ``/etc/hosts`` file which maps
  that IP to a domain name understood by the apps, eg:
  ``10.0.0.100    admin.perfplat.dev``
- Now you can access ie ``http://admin.perfplat.dev``
- Note that the hostname you choose in the ``/etc/hosts`` file affects the
  routing to the app: if you see "Bad Gateway" messages it's a clue that
  your hostname could be wrong.
