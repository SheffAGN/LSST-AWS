## So you want to run the LSST stack on an AWS cluster?

Cloud computing services like Amazon Web Services (AWS) and Google Cloud provide access to almost unlimited computing resources without the need to invest in large amounts of hardware. Using services like AWS's ParallelCluster, multiple cloud-computing nodes can be linked together to behave as a traditional High Performance Computing (HPC) cluster that use multi-processor standards that are familiar to many researchers (e.g., OpenMPI, NFS, Slurm, etc.).

With many aspects of the LSST stack already optimised for multi-processor systems, and having been inspired by a presentation by Hsin-Fang Chiang and Nate Lust at the 2019 LSST Project and Community Workshop, I thought I'd have a go at setting up an AWS cluster and running the LSST stack on it. While eventually successful, the process wasn't entirely straightforward, so I decided to document what I did for future reference and to assist others who also may want to run the LSST stack on an AWS HPC cluster.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/SheffAGN/LSSTwAWS/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
