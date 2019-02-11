# The Ansible "One Role to Rule them all" setup

I am a long time [Ansible](http://www.ansible.com) user and contributor
(since 2012) and I have been struggling with a decent setup for
a multi-environment case. I have been designing and re-designing a lot,
until I came up with this design. And what a coincidence, a customer
wanted a setup that was exactly this. So this concept is a real world
setup, working in a production environment.

The setup is (really) easy. At least, that is what I think.

It is, in fact, just one repository in Git that has control over all
Ansible roles used in the complete environment. This role is called
`setup`.

The `setup` repository contains a couple of things:

- The Ansible configuration
- The complete inventory split into the different environments
- All needed variables per environment, functionality or per host
- Small playbooks to run per environment
- Task lists that contain all tasks needed to get things done
- The `roles.yml`, containing all the roles we build ourself
- The `galaxy.yml`, containing all the roles we use from the Galaxy
- All (small) scripts to make things tick


## Configuration

Please, change the `config` file in the top-directory so it reflects
your own situation.

## Documentation, add-on and Caveat

This is the real **One Role that rules them all**.
This repo contains everything to control all the others.

All further documentation is in the `docs` repository.

