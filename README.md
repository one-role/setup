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

### The inventory

When looking at the inventory directory a little closer, there are
a couple of files per environment. For instance, the `dev` environment
contains three files `mysql`, `wiki` and `zz_groups`.

The `mysql` file contains the `dev_mysql` group, which contains all the
hosts that are responsible for running MySQL in the development
environment.

    [dev_mysql]
    db01.dev.example.net

And similar the `wiki` file contains all machines that run the Wiki
software in the development environment.

    [dev_wiki]
    wiki.dev.example.net

But the biggest "trick" in the `dev` environment are the files called
`zz_groups`. These contain the `dev` group, with all the child groups.

    [dev:children]
    dev_mysql
    dev_wiki

The reason this file is called `zz_groups` is because of the parsing
order of Ansible. When the Ansible inventory is a directory, Ansible
collects all files in the directory and processes them in alphabetical
order. *But*: A group can only be used as a _child_ if it is already
defined. So the definition of the `dev` group, containing all children
should come last, hence the name `zz_groups`.

In the top-level of the inventory there is a `zz_groups` file as well,
containing

    [mysql:children]
    dev_mysql
    tst_mysql
    acc_mysql
    prd_mysql
    
    [wiki:children]
    dev_wiki
    tst_wiki
    acc_wiki
    prd_wiki

This way you can run Ansible for the group `wiki`, `mysql` or the
subgroups `dev` or `dev_mysql`, giving very fine-grained control over
the set of hosts involved in the run.

![Inventory layout](/images/or_inventory.png)

### The variables

For the variables the standard setup is used. Variables are defined per
group, but we use a directory per group, instead of a file. All files in
such a group directory are merged together by Ansible, ensuring all
variables are available. The only reason to split it into separate files
is for maintainability, every item has its file, e.g.
`inventory/group_vars/dev/mysql_users`:

    ---
    # Users for geerlingguy.mysql role

    mysql_users:
      - name: "{{ nagios_mysql_user }}"
        host: 'monitoring.%.example.net'
        password: "{{ nagios_mysql_password }}"
        priv: "mysql.*:SELECT"
      - name: localweb-admin
        host: '192.168.0.%'
        password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6234356232393063..........03635383731353337363664393930
          3963313531613334..........53862306662363863323565646434
          6435646338386130..........66365363635306266326161353133
          3335646439303032..........33232303864356138656436643738
          6232
        priv: '*.*:SELECT/*.*:CREATE VIEW/*.*:INSERT'

Of course all defaults are in the `all` group directory, because all
hosts in the inventory are automatically member of the `all` group.

![Group variables](/images/or_group_vars.png)

### The roles

When the inventory and the variables are in place, all the involved
roles are need to be setup. Every role should have a branch for each
environment, so a `dev` branch for the `dev` environment and so on. And
all needed roles should be defined in the `roles.yml` file, like so:

    - src: https://github.com/one-role/apache.git
      scm: git
      version: master
      name: apache

The `version:` tag *should* be present, but the value is ignored, it
will be replaced by the `refresh` script with the branch that is
currently being checked out.

### Putting it all together

Once all the hosts, variables, roles and roles-files are in place, the
next thing to do is to roll it onto the Ansible control-node.

The order is:

    $ ssh root@ansible.example.net
    # cd /etc
    # mv ansible ansible.org
    # git clone https://git.example.net/one-role/setup ansible
    # cd ansible
    # ./refresh -f

Of course, after the initial checkout, the only command needed is the
`refresh` command. This will ensure all environments are filled with the
correct information.

Now run Ansible through `ansible_run` for the selected environment and
watch the magic happen.

![All together](/images/or_all.png)

## Documentation, add-on and Caveat

This is the real **One Role that rules them all**.
This repo contains everything to control all the others.

All further documentation is in the `docs` repository.

### More info

I presented this concept in the Ansible Room at
[CfgMgmtCamp](https://cfgmgmtcamp.eu/) on February 5th in Ghent,
Belgium. The slide deck is available at
[SpeakerDeck](https://speakerdeck.com/tonk/ansible-in-a-dec-tst-acc-and-prod-enviroment).

