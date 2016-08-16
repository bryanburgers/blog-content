I'm just getting started with [Vagrant][vagrant]. So far I'm very impressed.

I'm also just getting started with [Ansible][ansible]. Also, very impressed.

Vagrant has this handy feature called [provisioning][provisioning] that will set
up a box just the way you want it when it first starts up. And it includes
Ansible as one of the ways to provision a box. Let me tell ya, I keep being
impressed.

This works great for the Ubuntu 14.04 box ([ubuntu/trusty64 on Atlas][trusty64])
that I was provisioning. Getting a trusty64 box up and running is as easy as:

1. `vagrant up`
1. _Ready to go!_

Then I tried to do the same thing on an Ubuntu 16.04 box ([ubuntu/xenial64 on
Atlas][xenial64]). But the provisioning script would always fail because Python –
a requirement for Ansible – is not installed on the xenial64 box. So I ended up
having to manually install Python before provisioning the box. So many more
annoying little steps!

1. `vagrant up`. Wait for it to tell me provisioning failed. I **know**, alright!
1. `vagrant ssh`
1. `sudo apt-get install python`. Confirm **yes** this is what I want to do.
   Wait a while.
1. `exit`
1. `vagrant provision`
1. _Ready to go!_

It turns out, there's a solution! Vagrant lets you specify multiple provision
commands in one Vagrantfile, so Vagrant can be told to install Python first and
then use the Ansible script.

```
Vagrant.configure("2") do |config|
  # All of the other stuff

  # First, install python
  config.vm.provision "shell" do |s|
    s.inline = "apt-get install -y python"
  end

  # Then run the ansible script
  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.playbook = "/home/bryan/..."
  end
end
```

That's handy! Now Vagrant will handle booting and provisiong a xenial64 box as
nicely as a trusty64 box.

1. `vagrant up`
2. _Ready to go!_

[vagrant]: https://www.vagrantup.com/
[ansible]: https://www.ansible.com/
[provisioning]: https://www.vagrantup.com/docs/provisioning/
[trusty64]: https://vagrantcloud.com/ubuntu/boxes/trusty64
[xenial64]: https://vagrantcloud.com/ubuntu/boxes/xenial64
