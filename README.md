# GEDI (Genome Disassembler)
Genome scaffolding can be a powerful resource when creating a draft genome from assembled contigs. But how can we *test* scaffolders when contig information is scarce, sporadic and unreliable? <ins>GEDI (Genome Disassembler) takes a reference (possibly closed) genome and returns a set of artificial assemblies in the form of contigs, using real datasets as templates</ins>.  
Allow me to explain briefly: we downloaded all the representative genomes for which at least four assemblies in the form of contigs were present and stored the size information of both in a condensed JSON file (roughly 19Mb). When the user inputs a reference genome in the FASTA format, regardless of the taxonomic affiliation and only using the genome fraction that each chromosome contributes to, the program scans the real dataset and finds the representative that, structurally, most resembles the input.  
While my group and I are currently working on collecting the markers for a more accurate identification that includes a phylogenetic component (which would make more sense biologically), still we obtained interesting results when we tested this dataset scanning function. Please bear with us for a while longer, while we introduce the usage, tests that were performed, and potential applications. Thank you!


                ________________  ____
               / ____/ ____/ __ \/  _/
              / / __/ __/ / / / // /
             / /_/ / /___/ /_/ // /
             \____/_____/_____/___/
              T H E    G E N O M E
             D I S A S S E M B L E R
                       .-.
                      |_:_|
                     /(_Y_)\
                    ( \/M\/ )
_(Software by Christopher Riccardi, Art by Shanaka Dias)_
### Usage
GEDI is written in Python, and it runs on every python-supporting machine. We recommend calling it as ```python gedi``` on a Windows command prompt. On Linux kernel and MAC OSX you can get away with a ```chmod +x; ./gedi```. You know what I'm talking about. It doesn't require a proper installation, but you <ins>need to edit</ins> the line where it says ```dataset = loadDataset("dataset.json")``` and add the full path of where the /dataset.json resides on your PC.  
When providede with the right arguments, it produces a verbose console log and populates an output directory with a folder for each artificial contig. Inside each folder there will be three files: a FASTA file containing the actual artificial assembly, an oracle file with information on who maps where, and the template data that was used to produce that assembly.
```
usage: gedi [-h] [-o [OUTPUT]] [-r [RANDOM_SEED]] [-d [DOWNSIZE]] [-s [SPACER]] [-i [INVERSION_RATE]] [-m [MIN]] Reference_genome

Slice a reference genome into contigs using empirical distributions

positional arguments:
  Reference_genome

optional arguments:
  -h, --help            show this help message and exit
  -o [OUTPUT], --output [OUTPUT]
                        Output directory file name. Highly recommended. Default is an ugly timestamp.
  -r [RANDOM_SEED], --random_seed [RANDOM_SEED]
                        Random seed for shuffling vectors. 0 = do not shuffle.
  -d [DOWNSIZE], --downsize [DOWNSIZE]
                        Rescale reference genome size so that a virtual lower quote is considered. Default is to try using 95% of the genome.
  -s [SPACER], --spacer [SPACER]
                        Use at least N% of the ith chromosome as spacer between artificial contigs. Default is 0.02, or 2% of the ith contig as spacer.
  -i [INVERSION_RATE], --inversion_rate [INVERSION_RATE]
                        Rate of inversions per artificial assembly produced. Default is 0.4, i.e., invert 40% of the contigs produced.
  -m [MIN], --min [MIN]
                        Minimum required contig size. Default is 500 nucleotides.
```

### The Dataset and Benchmarking
The dataset consists of 774 representative genomes and 5235 assemblies. GEDI looks for the representative that is more similar to the user input FASTA and uses its assemblies as templates to build empirical-based artificial assemblies. Unitedly with the templates there are a number of parameters one can tune to obtain different assemblies; but the templates are the starting point. In this section we explain how the dataset search is implemented and how well we can expect it to correctly associate an entry.

#### A quick look at the dataset
To show how the search function works, we will first illustrate the most important features of the dataset.json file. Let's look at the very first record (i.e., with index '0' in a python interpreter):
```
>>> import json
>>> f = open("dataset.json")
>>> data = json.load(f)
>>> data['references'][0]['reference_accession']
'GCA_001708105.1'
```

The assembly submitted to the NCBI by the Massachusetts Institute of Technology is now the representative genome of _Komagataella pastoris_, an important yeast with many biotechnological applications. For future software expansion, we also included a 'superkingdom' key for every entry:
```
>>> data['references'][0]['superkingdom']
'Eukaryota'
```

But the most important feature is the 'ratios' or genome representation fractions. Each chromosome contributes to a percentage of the assembled genome, and we sought to exploit this ratio to identify sequences with similar fractions distributions. Last query example shows how to retrieve the ratios for _Komagataella pastoris_:  
```
>>> data['references'][0]['ratios']
[16.436, 19.052, 27.998, 34.646]
f.close() ## Let's not forget :)
```

Note that these are rounded percentages that don't need to add up to 100. Internally, the dataset also stores the absolute values so that if this software will be proved valid, future expansions can improve accuracy and applicability.

#### The search procedure
The current implementation uses a simple subtraction of padded ratio vectors, implemented as:  
```
def calcDistance(vec1, vec2):
    ## Called in a loop
    return sum(abs(vec1 - vec2))
```
The arguments vec1 and vec2 represent the genome representation fractions of the input FASTA (vec1) and each representative genome (vec2). Then GEDI finds the minimum of these distance values and proceeds accordingly.

#### Benchmarking
We tested the distance formula on the dataset in an all-against-all matrix after removing entries having the largest fraction > 0.9 (basically bacteria that were overpowering the signal). This is what we obtained:  

![rect17875](https://user-images.githubusercontent.com/69002653/184020694-b5f4b42c-8737-416e-a444-317697490318.png)

Except for a few samples (<1%) that contaminate the prokaryotes clusters and vice versa, there is virtually total distinction between the two superkingdoms included in the dataset. Intuitively, prokaryotes cluster together and are more similar to each other in their ratios than eukaryotes are with other eukaryotes. This is expected and it also means that when given a bacterial input it's possible that a phylogenetically distant candidate scores the best match. The dataset also stores the gc content of each representative genome, therefore it will be possible in the near future to refine the match and weigh the search towards more compositionally similar entries. We'd like to point out that a dynamic programming algorithm was also developed for a global alignment of the ratio values, with gap penalties and all. Didn't change much the results, it even performed worse. Apparently using the distance in the most strighforward way possible provides a good descriptor for distinguishing the superkingdoms.  

### FAQ  
*Why can't I find my reference/representative genome in the dataset.json file? The NCBI says it's representative!*  
We built the dataset using a number of criteria, which not only include the NCBI labelling "reference" or "representative". After gathering all the reference/representative genomes, we collected all the information for the NON reference/representative assemblies in the form of "contig" representation connected to the same taxid as the reference/representative. Then calculated the median N50, and filtered out everything that was below the median. This was performed to have a clean dataset, which could still include non-optimal assemblies. Then up to the first 9 assemblies (minimum of three) were selected for each reference/representative, therefore we automatically excluded reference/representative genomes with insufficient information at "contig" representation level. The whole point of GEDI is to use existing templates as an empirical distribution of ratios, hence the need to have at least some assembled, non-scaffolded genomes. 
    
_to be continued . . ._
