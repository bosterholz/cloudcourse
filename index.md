## Workflow management systems

### What have we done so far?

Last time we set up our own instance in the de.NBI cloud.
We created volumes and mounted them to get a mean to store our data in a persistent fashion.
The Openstack CLI was installed and used to get access to the cloud object storage, another means to store data in the cloud.
At the end we used the bibigrid tool to set up our own compute cluster.
This is where we want to continue today and use the newly started cluster to test different workflow management systems.

### Nextflow

The first candidate we will test is called [Nextflow](https://www.nextflow.io/). If you have questions or are stuck during the exercises, just look
at the [documentation](https://www.nextflow.io/docs/latest/index.html). The documentation is well written and can solve many questions quickly.

#### Lets get started with Nextflow

At first we will prepare a workplace we can use. Since we will work and share data between all instances of our cluster, we need to use the only 
directory which is accessible from all instances: `/vol/spool`

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

#### Say hello!
Lets test our new installation. What could be better than a good old "hello world!" ?

```bash
./nextflow run hello
```
Shorter than you expected? Yes, because we are a lazy bunch and cheated a little bit. Without a "hello" script in the same directory,
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
It should look like this:

```groovy
N E X T F L O W  ~  version 21.04.1
Launching `nextflow-io/hello` [boring_wilson] - revision: e6d9427e5b [master]
executor >  local (4)
[a5/b5f3bb] process > sayHello (1) [100%] 4 of 4 ✔
Hola world!

Ciao world!

Hello world!

Bonjour world!
```
Great! But as you can see this script was executed locally. We have multiple jobs and a whole cluster we could execute them on. Can't we share our jobs 
between all our instances using the workload manager "Slurm" we installed with our cluster? Sure, we just have to tell Nextflow to use Slurm instead 
of a local code execution. 

```bash
echo 'process.executor = "slurm"' > nextflow.config
```

With this we created the `nextflow.config` file. Nextflow will look for it during each run and use it to configure each run.
In this case we just told Nextflow to use Slurm as its primary executor. 

#### Let's BLAST something  

But a simple "hello world!" is just to boring. Let's do something more useful. We will take a quick look at [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi).

The "Basic Local Alignment Search Tool" or BLAST is a bioinformatics tool to find regions of similarity between biological sequences. 
The program compares nucleotide or protein sequences to sequence databases and calculates the statistical significance.

We will later download a database containing different antibiotic resistance genes. If an organism contains one or multiple of these genes 
it is generally concerning. In case of an infection with this organism, the choice of potential treatment options is severely limited due to these resistances. 

Another file with organisms will be downloaded. We will use BLAST to tell us which organisms contains antibiotic resistance genes from our database and need to be looked at more closely. 

But first things first. We need BLAST. We could install it locally on every instance in our cluster. But this would be a little bit tedious. We will use Nextflows build in support for the [Anaconda](https://www.anaconda.com/products/individual) package manager. You need to define which packages are needed for every single process, Nextflow will do the rest. If a package is missing on one of the instances Nextflow will fetch and install it for you. Just try it with BLAST. Open your favorite editor in your shell and create a `blast.nf` file with this content:

```groovy
#!/usr/bin/env nextflow

process blast {
  conda 'bioconda::blast'
  echo true

  '''
  blastp -h    
  '''
}
```
On running this script Nextflow will use Anaconda and install BLAST. Then BLAST will be called and prompt its help page. 

Perfect, since BLAST is up and running the only missing parts are the database and input organisms. We will quickly solve this.

```bash
cd /vol/spool/cloud_computing
mkdir input database
wget -O input/clostridium_botulinum.fasta https://raw.githubusercontent.com/bosterholz/MeRaGENE/dev/data/test_data/genome/clostridium_botulinum.fasta
cd database
git clone https://github.com/bosterholz/MeRaGENE.git
cd MeRaGENE
git checkout -b remotes/origin/dev
cd ..
mv MeRaGENE/data/databases/resFinderDB_23082018/ .
rm -rf MeRaGENE/
cd /vol/spool/cloud_computing/nextflow
```
Since everything is ready, can you build a working BLAST script?

