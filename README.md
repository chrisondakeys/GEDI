# GEDI (GEnome DIsassembler)
## Slice a reference genome into contigs using empirical distributions
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
Software by Christopher Riccardi, Art by Shanaka Dias
