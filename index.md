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

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/bosterholz/cloudcourse/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
