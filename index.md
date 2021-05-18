## Workflow managment systems

### What have we done so far?

Last time we set up our own instance in the de.NBI cloud.
We created volumes and mounted them to get a mean to store our data in a persistant fashion.
The Openstack CLI was installed and used to get access to the cloud object storage, another means to store data in the cloud.
At the end we used the bibigrid tool to set up our own compute cluster.
This is where we want to continue today and use the newly started cluster to test different workflow managment systems.

### Nextflow

The first candidate we will test is called [Nextflow](https://www.nextflow.io/). If you have questions or are stuck during the execises, just look
at the [documentation](https://www.nextflow.io/docs/latest/index.html). The documentation is well written and can solve many questions quickly.

#### Lets get started with Nextflow

At first we will prepare a workplace we can use. Since we will work and share data between all instances of our cluster, we need to use the only 
directory wich is accessible from all instances: `/vol/spool`

```shell
cd /vol/spool
mkdir -p cloud_computing/nextflow
cd cloud_computing/nextflow
```
With this we created a directory `cloud_computing` which will be used to gather all our experiments. Simultaneously a directory `nextflow`
was created and we jumped into it. Now we just need to install Nextflow locally and we are good to go.

```bash
curl -s https://get.nextflow.io | bash
```
If everything went well you should see a prompt with the current version number and some additional informations.
Here an example. Your version number should be equal or higher.

```
      N E X T F L O W
      version 21.04.1 build 5556
      created 14-05-2021 15:20 UTC 
      cite doi:10.1038/nbt.3820
      http://nextflow.io


Nextflow installation completed. Please note:
- the executable file `nextflow` has been created in the folder: /raid1/benedikt/cloud_computing/nextflow
- you may complete the installation by moving it to a directory in your $PATH
```
Lets test our new installation. What could be better than a good old "hello world!" ?

```bash
./nextflow run hello
```
Shorter than you expected? Yes, because we are a lazzy bunch and cheated a little bit. Without a "hello" script in the same directory,
Nextflow looked for a corresponding script on its github account. There it found something, checked it out using the integrated git module and started it locally. What you see are the results for running this script:

```groovy
#!/usr/bin/env nextflow

cheers = Channel.from 'Bonjour', 'Ciao', 'Hello', 'Hola'

process sayHello {
  echo true
  input: 
    val x from cheers
  script:
    """
    echo '$x world!'
    """
}

```

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/bosterholz/cloudcourse/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
