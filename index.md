## Workflow management systems

### What have we done so far?

Last time we set up our own instance in the de.NBI cloud.
We created volumes and mounted them to get an option to store our data in a persistent fashion.
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
Nextflow looked for a corresponding script on its github account. There it found something, cloned it using the integrated git module and started it locally. What you see are the results for running this script:

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
it is generally concerning. In case of an infection with this organism, the choice of potential treatment options could be severely limited due to these resistances. 

Another file with organisms will be downloaded. We will use BLAST to tell us which organisms contain antibiotic resistance genes from our database and need to be looked at more closely. 

But first things first. We need BLAST. We could install it locally on every instance in our cluster. But this would be a little bit tedious. We will use Nextflows build in support for the [Anaconda](https://www.anaconda.com/products/individual) package manager. You need to define which packages are needed for every single process, Nextflow will do the rest. If a package is missing on one of the instances Nextflow will fetch and install it for you. Just try it with BLAST. Open your favorite editor in your shell and create a `blast.nf` file with this content:

```groovy
#!/usr/bin/env nextflow

process blast {
  conda 'bioconda::blast'
  echo true

  """
  blastp -h    
  """
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
git checkout dev
cd ..
mv MeRaGENE/data/databases/resFinderDB_23082018/ .
rm -rf MeRaGENE/
rm resFinderDB_23082018/*_NA*
cd /vol/spool/cloud_computing/nextflow
```
You can now build your own BLAST pipeline using Nextflow. Let me give you some tipps, so that you can get a head start.

Here are some important Nextflow code snippets you will need:

[How do I use files or a whole folder as input?](https://www.nextflow.io/docs/latest/process.html#input-of-files)

[We have multiple database files, how do we use EACH of them?](https://www.nextflow.io/docs/latest/process.html#inputs)

[How do I handle my output?](https://www.nextflow.io/docs/latest/process.html#output-files)

[How do I get my output in a file?](https://www.nextflow.io/docs/latest/operator.html?highlight=collect#collectfile)

Here a blast call template:
```
blastx -db YOUR-DATABASE-FSA-FILE -query YOUR-INPUT-FASTA -out NAME-OF-YOUR-OUTPUT
```
Whith this you should be good to go! Good luck!

### Sankemake

After we got a quick glimpse on how to handle Nextflow, we will test our secound candidate today: [Snakemake](https://snakemake.github.io/)

#### Setup

Let's set it up: 
```
cd /vol/spool/cloud_computing
mkdir snakemake
cd snakemake/
wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh
bash Mambaforge-Linux-x86_64.sh
```
You will see a dialogue during the installation of Snakemake. Press ENTER, answear the license agreament with YES.
Then change the installation path to: `/vol/spool/cloud_computing/snakemake/mamba`. We don't want to initialise conda so we will type NO.
Now we need the tutorial data, we download and extract everything with: 

```
wget https://github.com/snakemake/snakemake-tutorial-data/archive/v5.24.1.tar.gz
tar --wildcards -xf v5.24.1.tar.gz --strip 1 "*/data" "*/environment.yaml"
```
After a quick introduction into Snakemake we will try to replicate our BLAST example. To make things easier we add BLAST to our downloaded `environment.yml`. These are packages which will be installed at the start of a new initialisation. So we will start with an installed BLAST and don't need to add it later on. 

Again, open the `environment.yml` with your favorite editor and add `- blast =2.11` at the beginning of your package list.
Now we initialize our environment:
```
source /vol/spool/cloud_computing/snakemake/mamba/bin/activate base
mamba env create --name snakemake --file environment.yaml
source /vol/spool/cloud_computing/snakemake/mamba/bin/activate snakemake
```
#### Let's start with Snakemake

You should now be able to use the `Snakemake` command in your bash. It would be a little bit boring to start with `hello world` for the second time, so we will look at an easy but real bioinformatic tool call. Create a file called `snakefile` and fill it with:

```
rule bwa_map:
    input:
        "data/genome.fa",
        "data/samples/A.fastq"
    output:
        "mapped_reads/A.bam"
    shell:
        "bwa mem {input} | samtools view -Sb - > {output}"
```
All the data we need we downloaded in the previous steps. We can immediatly start calling Snakemake. In Snakemake we start with a so called dry run.
Snakemake will look if everything is in order and print every paths it will create or tell us of problems it encountered. 

`snakemake -np mapped_reads/A.bam`

Everything should be okay. We start the real run with:

`snakemake --cores 1 mapped_reads/A.bam`

This was a real static run. Is there a way to do it in a more dynamic fashion? Yes, of course. Just change your `snakefile` like you see below.

```
rule bwa_map:
    input:
        "data/genome.fa",
        "data/samples/{sample}.fastq"
    output:
        "mapped_reads/{sample}.bam"
    shell:
        "bwa mem {input} | samtools view -Sb - > {output}"
```
Now try these dry runs:

`snakemake -np mapped_reads/B.bam`

`snakemake -np mapped_reads/A.bam mapped_reads/B.bam`

`snakemake -np mapped_reads/{A,B}.bam`

Do you see what happens? Try running these for real. Are there Problems? 

You should now know enough to replicate our Nextflow BLAST call with Snakemake. Try it!


### Common Workflow Language (CWL)
CWL is not a software, but rather a standard for describing data analysis workflows. A range of different workflow engines can be used to execute CWL-workflows. We call these workflow engines “cwl-runners”, but some of them are compatible to other workflow languages as well.

#### Basic Concepts
CWL is used to describe two types of document: *CommandLineTool* and *Workflow*.
A *CommandLineTool* describes a single tool that can be invoked from the command line, like `echo` or `blast`. Based on this description, a cwl-runner is able to execute the tool.
A *Workflow* is used to link multiple CommandLineTools (or other workflows), describing the flow of data between them.
A cwl-runner also needs to know which input to feed into a tool or workflow. This is done with a job file, where inputs are described in the YAML-format.

#### Toil 
For the purpose of executing our CWL code, we will be using the [Toil ](https://toil.readthedocs.io/en/latest/) workflow engine. To install Toil (and blast) in a virtual environment, we will use a conda. Toil has several optional features, and the ability to interpret CWL code is one of them. So we have to specify that we want to install the cwl-feature when wie install Toil.


```bash
cd /vol/spool/cloud_computing
mkdir miniconda
cd miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```
Accept the license (scroll through the text with ENTER, then type `yes`) and specify the installation path as `/vol/spool/cloud_computing/miniconda/inst`. Type `no` when asked about conda init. We will now activate our conda virtual environment, create another virtual environment for our work with CWL and install toil and blast.

```bash
source inst/bin/activate
conda create --name cwl
conda activate cwl
conda install pip
pip install toil[cwl] boto boto3
conda install -c bioconda blast
```




#### Running a CWL Workflow
We need a piece of CWL code for toil to execute. Once again, we will work in `/vol/spool` so data will be available across all nodes.
```bash
cd /vol/spool
mkdir -p cloud_computing/cwl
cd cloud_computing/cwl
```

Paste the following into a text file named `exampleTool.cwl`:
```
#!/usr/bin/env cwl-runner

cwlVersion: v1.2                 #Toil can handle different versions of the CWL-specification.
class: CommandLineTool
baseCommand: echo

requirements:
  InlineJavascriptRequirement: {}

inputs:
  message:                       #Name of our first (and only) input
    type: string                 #The type of the input
    inputBinding:
      position: 1                #Determines the order of inputs on the command line

outputs:                         #Here we tell the CommandLineTool which outputs to collect
  answer:                        #The internal name of our first (and only) output file
    type: stdout                 #Echo writes to stdout, so we have to capture standard output 
    
stdout: $(inputs.message).txt    #The standard output we have captured will be written to a file.
                                 #(in this case, the input string with the suffix ".txt")
```
Paste the following into a text file named `exampleJob.yml`:
```
message: "Hello"
message2: "Hallo"
message3: "Bonjour"
message4: "Hola"
```
Now execute the CWL code on the local machine using Toil:
```bash
mkdir myWorkdir
toil-cwl-runner --workDir myWorkdir --outdir myOutputs --clean always exampleTool.cwl exampleJob.yml
```

Toil will run the code in example.cwl, using the input specified in `exampleJob.yml` and save the results to myOutputs. We specified four different input strings in our `exampleJob.yml`, but `exampleTool.cwl` will only use the input specified behind `message`. Your output should turn up in the `myOutputs` directory.


#### Running a Workflow
Lets save a multi-step CWL-workflow into a file `exampleWorkflow.cwl` so Toil has multiple jobs to run.
```
#!/usr/bin/env cwl-runner

cwlVersion: v1.2
class: Workflow

inputs:                          #The different inputs to our workflow, each has a name and a type (string)
  message: string                
  message2: string
  message3: string
  message4: string

steps:                           #A list of steps the workflow has to carry out. Toil will decide the order of steps.
  print1:                        #The internal name of the first step
    run: exampleTool.cwl         #The CommandLineTool that we will run in the first step
    in:                          #The input that the workflow will feed into the first step
      message: message           
    out: [answer]
  print2:
    run: exampleTool.cwl
    in:
      message: message2          #Our CommandLineTool expects an input for the "message" parameter. We provide the "message2" string from our workflow inputs
    out: [answer] 
  print3:
    run: exampleTool.cwl
    in:
      message: message3
    out: [answer]                #A list of outputs that the workflow will collect from the CommandLineTool. We specified the name "answer" in the CommandLineTool above
  print4:
    run: exampleTool.cwl
    in:
      message: message4
    out: [answer] 
    
outputs:                         #The outputs our workflow will create
  text1:                         #The internal name of the first of four outputs
    type: File                   #The type of the output. We know that our CommandLineTool will create a file
    outputSource: print1/answer  #Where the fetch the output (in this case from the print1 step). Answer is the internal name we have given to the output earlier
  text2:
    type: File
    outputSource: print2/answer
  text3:
    type: File
    outputSource: print3/answer
  text4:
    type: File
    outputSource: print4/answer
    
```
Now submit the job to your slurm cluster like this:
```bash
toil-cwl-runner --batchSystem slurm --disableCaching --jobStore myJobstore --workDir myWorkdir --outdir myOutputs --clean always exampleWorkflow.cwl exampleJob.yml
```
Again, all your outputs should show up in `myOutputs`

#### Blast Tool
Like before we want to use blast to look for antibiotic resistances in the *Clostridium botulinum* genome. The genome will be our blast query, the individual antibiotic genes are the databases we want to search. Doing this on the command line for a single resistance gene might look like this:

```bash
blastx -query /vol/spool/cloud_computing/input/clostridium_botulinum.fasta -db /vol/spool/cloud_computing/resFinderDB_23082018/tetracycline_NA.fsa
```
Try to create a CWL CommandLineTool that can be used for this kind of blast search. It should be able to process the following job file:
```
target_fasta:
  class: File
  path: /vol/spool/cloud_computing/input/clostridium_botulinum.fasta
query_fasta:
  class: File
  path: /vol/spool/cloud_computing/resFinderDB_23082018/tetracycline_AA.fsa
```
For this task, it is important to know that the blast database for the tetracycline sequence actually consists of four files:
- tetracycline_NA.fsa
- tetracycline_NA.fsa.phr
- tetracycline_NA.fsa.pin
- tetracycline_NA.fsa.psq

Our CWL runner needs to know that files 2, 3 and 4 contain information pertaining to our primary file and blast expects to find them in the same directory. For this reason, we need to specify the `secondaryFiles` property for our input parameter.
```
someInputParameter:
  type: File
  inputBinding:
    position: 5
    prefix: -somePrefix
  secondaryFiles
  - .phr
  - .pin
  - .psq
```
This tells the CWL runner to look for Files that have the same basename as the input parameter, but end in .phr, .pin or .psq.<br>
The [CWL User Guide](https://www.commonwl.org/user_guide/) provides an excellent introduction into writing CWL and you may also use it to look up individual topics.<br>
When you have written your CommandLineTool, try using toil to run it with the job file that was specified in this section.

#### Docker
Using docker images to run our tools in containers solves a lot of problems for us. The first advantage is that we do not have to install software (like blast) on every node that processes our jobs. It also makes our workflows more robust and analyses will be easier to reproduce, since the software will always be carried out in an environment that is exactly identical. You can add the following code to your CommandLineTool to use docker.
```
hints:
  DockerRequirement:
    dockerPull: ncbi/blast:2.11.0
```
Now, when you run this tool, the CWL-runner will test if docker is available on the system. If it is, the runner will download a docker image that can run version 2.11.0 of the blast software.

#### Blast Workflow
Look at the examples for [writing workflows](https://www.commonwl.org/user_guide/21-1st-workflow/index.html) and [scattering workflows](https://www.commonwl.org/user_guide/23-scatter-workflow/index.html) from the CWL User Guide. Can you build a workflow that uses your blast CommandLineTool to blast against multiple antibiotic resistance sequences? Use the following job file:
```
query_fasta:
  class: File
  path: /mnt/huge/cloud-kurs/input/clostridium_botulinum.fasta
database_array:
- {class: File, path: /vol/spool/cloud_computing/resFinderDB_23082018/aminoglycoside_AA.fsa }
- {class: File, path: /vol/spool/cloud_computing/resFinderDB_23082018/beta-lactam_AA.fsa }
- {class: File, path: /vol/spool/cloud_computing/cloud-kurs/resFinderDB_23082018/colistin_AA.fsa }
```
