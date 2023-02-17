# how_to_blast

## Installing blast on the cluster

1.) make sure you are in you-re logged into your cluster account
2.) type the following code:

module load bio 
module load megablast

* To check the version use this:

blast* -version (* is either blastn, blastp, etc)

## Getting started 

3.) Set up a directory called sunflower_genomics and then within it I made directories v1 and v2 and included the fasta genome files (HA412.v1.1.bronze.20141015.fasta and Ha412HOv2.0-20181130.fasta respectively).

Data can be found here: 

https://sunflowergenome.org/assembly-data/

4.) Then you need to make a local blast database for each reference that you want to work with (except XRQ -- I don't see much point in going from v1 to XRQ or v2 to XRQ). Making a blast db is very easy. Here are the commands I used:

makeblastdb -in v1/HA412.v1.1.bronze.20141015.fasta -dbtype nucl -out v1/v1db

makeblastdb -in v2/Ha412HOv2.0-20181130.fasta -dbtype nucl -out v2/v2db

## Comparing unkown data

Now suppose you found an enzyme on KEGG that includes the sunflower XRQ sequence (e.g. https://www.genome.jp/entry/han:110894586+han:110896060+han:110927165). Copy the nucleotide sequence and paste it into a text document using fasta format:
query.txt:
>query
ATGTCTGTTGCTCTGATTTGGGTTGTTTGCAATAGATTTGGATTCTTGGAGACCACAAAG\nTTTGAAACTTTATCAAAATCAAGAACCCTTTTGAGGAGTGAAAGAATCAAGAATCTTGGT\nAAGAAACATGAATGTAAATCCAGCTACATGAATGCAGATTTTATGGGATTGAATTATGGG

## Running blastn

If you want to find the sequence in v1, use:

blastn \
   -db v1/v1db \
   -query query.txt \
   -out query_resultsv1.txt

or if v2, use:

blastn \
   -db v2/v2db \
   -query query.txt \
   -out query_resultsv2.txt

* both .txt will appear in the main directory

Congrats! You've just did blastn

