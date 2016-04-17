
# Introduction to Rex

<http://preaction.github.io/Introduction-to-Rex/>

A short introduction to Rex, a Perl server automation tool, and
a gallery of Rex's features.

by Doug Bell ([preaction](http://preaction.me))  
<a href="http://twitter.com/preaction"><i class="fa fa-twitter"></i> @preaction</a>  
<a href="http://github.com/preaction"><i class="fa fa-github"></i> preaction</a>  
[Chicago.PM](http://chicago.pm.org)  

Source: <https://github.com/preaction/Introduction-to-Rex>  
License: [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)

------

# Rex is about Running Tasks

---

# `Rexfile` defines tasks

---

```perl
desc 'Check the disk free on the relevant drives';
task 'df' => sub {
    say scalar run 'df -H /usr*';
};
```

------

# `rex` runs tasks

---

```
$ rex df
[2015-04-08 12:17:39] INFO - Running task df on <local>
Filesystem             Size   Used  Avail Use% Mounted on
/dev/mapper/VGRoot-LogVolroot
                       5.3G   3.5G   1.6G  70% /
/dev/sde1               64G    41G    20G  68% /usr2
```

------

# Rex runs tasks on hosts

---

# `rex -H <host> <task>`

---

```
$ rex -H scsqhdadap01 df
[2015-04-08 12:18:54] INFO - Running task df on scsqhdadap01
[2015-04-08 12:18:54] INFO - Connecting to scsqhdadap01:22 (nbkyslo)
[2015-04-08 12:18:54] INFO - Connected to scsqhdadap01, trying to authenticate.
[2015-04-08 12:18:54] INFO - Successfully authenticated on scsqhdadap01.
Filesystem             Size   Used  Avail Use% Mounted on
/dev/mapper/volgrp01-root
                       8.4G   4.1G   3.9G  52% /
/dev/mapper/volgrp02-usr2
                       144G    65G    73G  47% /usr2
/dev/mapper/volgrp01-usr3
                       108G    73G    30G  72% /usr3
/dev/mapper/volgrp03-usr4
                       432G   203G   208G  50% /usr4
/dev/fioa              773G   419G   315G  58% /usr5
```

---

```
$ rex -H 'scsqhdapap[01..04]' df
```

------

# Rexfile configures host groups

---

```
group ops_prd => 'scsqhdapap0[1..4]';
group ops_uat => 'ltwqhdacap0[1..4]';
group ops_dev => 'scsqhdadap0[1..2]';
```

------

# Rex runs tasks on host groups

---

# `rex -g <group> <task>`

---

```
$ rex -g ops_dev df
[2015-04-08 12:23:21] INFO - Running task df on scsqhdadap01
[2015-04-08 12:23:21] INFO - Connecting to scsqhdadap01:22 (nbkyslo)
[2015-04-08 12:23:21] INFO - Connected to scsqhdadap01, trying to authenticate.
[2015-04-08 12:23:22] INFO - Successfully authenticated on scsqhdadap01.
Filesystem             Size   Used  Avail Use% Mounted on
/dev/mapper/volgrp01-root
                       8.4G   4.1G   3.9G  52% /
/dev/mapper/volgrp02-usr2
                       144G    65G    73G  47% /usr2
/dev/mapper/volgrp01-usr3
                       108G    73G    30G  72% /usr3
/dev/mapper/volgrp03-usr4
                       432G   203G   208G  50% /usr4
/dev/fioa              773G   419G   315G  58% /usr5
[2015-04-08 12:23:25] INFO - Running task df on scsqhdadap02
[2015-04-08 12:23:25] INFO - Connecting to scsqhdadap02:22 (nbkyslo)
[2015-04-08 12:23:27] INFO - Connected to scsqhdadap02, trying to authenticate.
[2015-04-08 12:23:30] INFO - Successfully authenticated on scsqhdadap02.
Filesystem             Size   Used  Avail Use% Mounted on
/dev/mapper/volgrp01-root
                       8.4G   1.8G   6.2G  23% /
/dev/mapper/volgrp02-usr2
                       144G    95G    42G  70% /usr2
/dev/mapper/volgrp01-usr3
                       108G    90G    13G  88% /usr3
/dev/mapper/volgrp03-usr4
                       432G   258G   153G  63% /usr4
```

------

# Rexfile configures environments

---

```
environment ops_dev => sub {
    group all => 'scsqhdadap0[1..2]';
    group reuters => 'scsqhdadap02';
    group db => 'scsqhdadap02';
};

environment ops_prd => sub {
    group all => 'scsqhdapap0[1..4]';
    group reuters => 'scsqhdapap04';
    group db => 'scsqhdapap01';
};
```

------

# Rex runs tasks in environments

---

# `rex -E <env> <task>`

---

```
$ rex -E ops_dev -g db df
[2015-04-08 12:35:31] INFO - Running task df on scsqhdadap02
[2015-04-08 12:35:31] INFO - Connecting to scsqhdadap02:22 (nbkyslo)
[2015-04-08 12:35:32] INFO - Connected to scsqhdadap02, trying to authenticate.
[2015-04-08 12:35:35] INFO - Successfully authenticated on scsqhdadap02.
Filesystem             Size   Used  Avail Use% Mounted on
/dev/mapper/volgrp01-root
                       8.4G   1.8G   6.2G  23% /
/dev/mapper/volgrp02-usr2
                       144G    95G    42G  70% /usr2
/dev/mapper/volgrp01-usr3
                       108G    90G    13G  88% /usr3
/dev/mapper/volgrp03-usr4
                       432G   257G   153G  63% /usr4
```

------

# Rexfile has configuration

---

```
set datlib => '/datlib/hist';
environment ops_uat => sub {
    set datlib => '/datlib/app1_dr';
};
```

---

```
desc 'Restart process controller';
task 'restart_pc' => sub {
    my $datlib = get 'datlib';
    my $env = environment;
    run "$datlib/pc/envs/$env/manager/bin/apache restart";
    run "$datlib/pc/envs/$env/manager/bin/boss.pl restart";
};
```

---

```
$ rex -E ops_dev -g pc restart_pc
```

------

# Tasks can have default groups

---

```
environment ops_dev => sub {
    group pc => 'scsqhdadap02';
};
environment ops_prd => sub {
    group pc => 'scsqhdapap01';
};
```

---

```
desc 'Restart process controller';
task 'restart_pc',
    groups => [qw( pc )],
    sub {
        my $datlib = get 'datlib';
        my $env = environment;
        run "$datlib/pc/envs/$env/manager/bin/apache restart";
        run "$datlib/pc/envs/$env/manager/bin/boss.pl restart";
    };
```

---

```
$ rex -E ops_dev restart_pc
```

------

# Tasks can run other tasks

---

```
desc "Update the PC repository";
task 'update_pc',
    group => [qw( pc )],
    sub {
        my $datlib = get 'datlib';
        run "cd $datlib/pc/versions/master && git pull";
        die "Task failed. Stopping." if $?;
        run_task 'restart_pc', on => connection->server;
    };
```

---

```
$ rex -E ops_dev update_pc
```

------

# Tasks can use parameters

---

```
desc "Install CPAN modules";
task 'cpan', sub {
    my ( $opt ) = @_;
    my $modules = join " ", split /[, ]/, $opt->{module};
    run "cpan $modules";
};
```

---

```
$ rex -g ops_dev cpan --module='Mojolicious Dancer'
```

------

# Rex can do more

------

# Rex can install files

---

```
task deploy_pc =>
    group => [qw( pc )],
    sub {
        file '/etc/apache2/httpd.conf',
            ensure => 'present',
            source => 'etc/httpd.conf',
            on_change => sub {
                run_task 'restart_pc', on => connection->server;
            };
    };
```

------

# Rex can generate files from templates

---

```
task deploy_pc =>
    group => [qw( pc )],
    sub {
        my $config = template(
            'etc/httpd.conf',
            server => connection->server,
        );

        file '/etc/apache2/httpd.conf',
            ensure => 'present',
            content => $config,
            on_change => sub {
                run_task 'restart_pc', on => connection->server;
            };
    };
```

------

# Rex can sync directories
## `rsync`

---

```
use Rex::Commands::Rsync;

task sync_www =>
    sub {
        sync 'www', '/var/www';
    };
```

------

# Rex has more configuration

## Rex::CMDB

---

# Configure by host

---

# Configure by environment

---

# Configure by OS

------

# Rex has plugins

<http://modules.rexify.org>

------

# It's over!

* [Rex](http://rexify.org)

