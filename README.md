fluo-dev
==========

Scripts and configuration designed to simplify the running of Fluo infrastructure
during development.  While these scripts make it easy to deploy Fluo, they can wipe
the underlying data off your cluster.  Therefore, they should not be used in production.
They are designed for developers who need to frequently upgrade Fluo, test their code,
and do not care about preserving data.

Installing fluo-dev
-------------------

First, clone the fluo-dev repo on a local disk with enough space to run Hadoop, Accumulo, etc:
```
git clone https://github.com/fluo-io/fluo-dev.git
```

Create `env.sh` from the example.  This file is used to configure fluo-dev for your enviornment.
```
cd conf/
cp env.sh.example env.sh
vim env.sh
```

If you have not already, clone the Fluo repo.  Set `FLUO_REPO` in your env.sh to this location.
```
git clone https://github.com/fluo-io/fluo.git
```

All commands are run using the `fluo-dev` script.  If want to be able to run this script from
any directory, you can optionally add the following to your `~/.bashrc`:
```
export PATH=/path/to/fluo-dev/bin:$PATH
```

You will need to invoke the `fluo` command using `fluo-dev fluo` as `fluo-dev` will set up
the correct Hadoop environment variables for the `fluo` command.  To avoid using `fluo-dev fluo`
every time, add the following alias to your `~/.bashrc`:
```
alias fluo='fluo-dev fluo'
```

Running Fluo dependencies
-------------------------

With fluo-dev set up, you can now use it to download, configure, and run Fluo and its dependencies.

First, run the command below to download the binary tarballs of Fluo dependencies (i.e Accumulo, Hadoop, 
and Zookeeper). The command below downloads all tarballs and their corresponding file hashes and 
signatures to the directory specified by `DOWNLOADS` in your env.sh. It will use the Apache download 
mirror specified by `APACHE_MIRROR` in env.sh.  Other mirrors can be chosen from [this website][1].
This command will also output hashes and signatures (if you have `gpg` installed) of the downloaded
software. It is important to inspect this output before installing the software.
```
fluo-dev download
```

Next, run the following command to install the downloaded packages to the path specifed by 
`SOFTWARE` in your env.sh.
```
fluo-dev install
```

Now, run the command below to copy Accumulo, Hadoop, and Zookeeper configuration in `fluo-dev/conf`
to their respective installs.  This command will create a `fluo.properties` file at 
`fluo-dev/conf/fluo.properties` that will be used by fluo when its deployed.
```
fluo-dev configure all
```

Reset all Fluo dependencies (i.e Hadoop, Accumulo, Zookeeper) which will clear any underlying
data and restart them (if they are running).
```
fluo-dev reset all
```

Confirm that everything started by checking the monitoring pages of Hadoop & Accumulo:
 * [Hadoop NameNode](http://localhost:50070/)
 * [Hadoop ResourceManager](http://localhost:8088/)
 * [Accumulo Monitor](http://localhost:50095/)

Running Fluo
------------

With its dependencies running, Fluo can now be started.  Before starting Fluo, you can optionally
verify or change configuration in the `fluo.properties` file generated by `fluo-dev configure`.  
```
vim conf/fluo/fluo.properties
```

If you want to run Fluo with observers, avoid configuring them in `fluo.properites`.  Instead, you
should create a file called `observer.props` in `conf/` by copying the example:
```
cp conf/observer.props.example conf/observer.props
vim conf/observer.props
```

The example `observer.props` file is configured to run the [fluo-stress][stress] example application.  
The observer jar for [fluo-stress][stress] can be obtained by cloning and building its repo using the
the steps below:
```
git clone https://github.com/fluo-io/fluo-stress.git
cd fluo-stress/
mvn package
ls target/fluo-stress-*.jar
```

Copy your observer JAR for your application to `conf/fluo/observers`.  All jars in this directory will
be include on the classpath when you deploy Fluo.
```
cp /path/to/observer.jar conf/fluo/observers/
```

Deploy Fluo using the command below which will remove any existing install, rebuild fluo, 
install fluo, and configure it using your configuration in `conf/fluo`:
```
fluo-dev deploy
```

Finally, confirm that Fluo is running:
```
fluo yarn status
```

The commands above are designed to be repeated.  If Hadoop or Accumulo become unstable, run
`fluo-dev reset all` and then `fluo-dev deploy` to reset Hadoop/Accumulo and redeploy Fluo.
If you make any code changes to Fluo and want to test them, run `fluo-dev deploy` which builds 
the latest in your cloned Fluo repo and deploys it.

Updating observers
------------------

If you want to update your obsever code, remove any old jars from `fluo-dev/conf/fluo/observers`
and copy your updated jar to the directory:

```
rm conf/fluo/observers/*.jar
cp /path/to/observer.jar conf/fluo/observers/
```

If it is OK to clear your cluster, run the command below to redeploy fluo:
```
fluo-dev deploy
```

If you want to save the data on your cluster, follow the commands below to update the observers in 
your deployment and start/stop fluo without losing data:
```
rm software/fluo-1.0.0-beta-1-SNAPSHOT/lib/observers/*.jar
cp /path/to/observer.jar software/fluo-1.0.0-beta-1-SNAPSHOT/lib/observers
fluo yarn stop
fluo yarn start
```

[1]: http://www.apache.org/dyn/closer.cgi
[stress]: https://github.com/fluo-io/fluo-stress
