This lab will use this fasta file: brca1.fasta

## We will use the Farm cluster for this assignment.

## Log in:
To log in, from a Unix terminal type your username in the command below:

ssh [your_username]@farm.cse.ucdavis.edu
Then enter your password. I highly recommend pasting your password into the terminal instead of typing it. Note: the backspace or delete keys do not work for your password, so if you make a mistake, hit Enter and then try again.

You will arrive at your personal home directory. Only you can write / modify files in this directory.

## Moving files on and off:
Open the FileZilla program. This program sets up a secure connection to the Farm file server and lets you move files on and off of the cluster. It works a lot like Windows File Explorer of Mac Finder.

To use:

1.) Select File > Site Manager. Create a New Site, enter: Host: farm.cse.ucdavis.edu, Protocol: SFTP, Logon Type: "Normal", User: username, Password: password, Click Connect.

2.) On the left is a file explorer for your computer. Navigate to your Desktop, or wherever you have downloaded the file brca1.fasta.

3.) On the right is a file explorer for Farm's file system. Create a directory called test. You can also do this in the terminal (mkdir Lab_7).

4.) Find the file: brca1.fasta, and drag it onto Farm.

## Running BLAST
The BLAST programs are installed on Farm. They are fairly intensive programs, requiring a moderate amount of RAM, and sometimes a lot of time to run.

We do not want to run BLAST on the head node! This could bog-down this computer and prevent others on the cluster from doing their work!

First, request an allocation on one of the compute nodes:

## Requesting an interactive job

srun -p bit150h -t 20 --pty bash -l

* srun is a command that asks for an allocation
* -p bit150h asks to use the bit150h partition. This means use the resources assigned to our class
* -t 20 asks for 20 minutes. After this time, your allocation will shut down
* --pty bash -l asks for the allocation to run Bash just as if you opened a new terminal
* Hit Enter, and then wait a minute. Eventually you will see some messages saying you are logged in to a job.

To see info on your job, type: squeue -u username

## The blast program
Now that we have access to a powerful computer, we can use BLAST to search a database for our sequence.

Our sequence is in the file brca1.fasta, the file we moved to Farm. View the file with:

head brca1.fasta

Note: you need to be inside your test folder to run this command, so move there now if you haven't (with the cd test command).

## The FASTA format

The FASTA format is a common format for storing sequence files. Each sequence has a header (containing all the metadata), and 1+ lines containing the sequence. The header is the first line, and begins with the > character. The rest of the line contains various annotations (a very compressed version of the GenBank format). Longer sequences are split over multiple lines. But the line breaks are irrelevant. Programs like BLAST know to keep reading the sequence until the end of the file.

Multiple sequences can be held in the same file. They are separated by header lines.

The blast program that we will use is blastp. You can check if the program is installed by typing its name and giving the -version argument:

blastp -version

The default version on Farm is 2.6.0+, built on Jan 15 2017. This is an old version of the progam. We want to use version 2.6.0+, built on Sept 13 2017. To get this, we have to tell Farm that we want a different version. We can do this by typing:

module load bio
blastp -version

The bio module includes many bioinformatics programs besides blast.

Now, we can run blast. A run looks like this:

blastp -db /group/bit150/blast_databases/refseq_human_protein -query brca1.fasta -out brca1_blast.txt

* blastp is the program for protein-protein blast.
* db /group/bit150/blast_databases/refseq_human_protein specifies the blast database that we want to use. We select the refseq_human_protein database in the /group/BIT150/blast_databases/directory
* query brca1.fasta specifies the query sequence
* out brca1_blast.txt specifes we want the output stored in the file called brca1_blast.txt

Look at the file. It's long.

Hint A good viewer is less. Type less brca1_blast.txt. You can scroll with the arrow keys, page down with space, and then quit less by typing q.

Or, you can download the file to your computer using FileZilla, and view it in Notepad++.

Let's modify the command. We can change the output style into a concise table. Rather than re-typing, use the up-arrow to get find the command. Note: you could also use history. Now, instead of moving the cursor with the arrow keys, copy it to a Notepad++ document, and edit it there.

For clarity, it's often nice to break up long commands into several lines. You can do this by putting a \ character at the end of each line. Note: be careful to be extrememly precise here. The has to be a space before the \ and no characters after the \ on that line.

blastp  \
   -db /group/bit150/blast_databases/refseq_human_protein \
   -query brca1.fasta \
   -out brca1_blast_table.txt \
   -outfmt 7   
I added the option -outfmt 7. This changes the format of the output to a table. See blastp -help for more formatting options.

You can copy this whole set of code and paste into the terminal. Now, look at this output file using less. This file is pretty wide; it has many columns. Files like this are better viewed with the -S option: less -S. Now you can scroll both up/down and left/right.

When you're done, exit less with q and then the compute node by typing:

exit

Always remember to exit your srun session when you are done - this frees up resources for others!

Notes:
* There are many more options for the blastp program. You can view them with blastp -help.
* There are many databases that you could blast against. You can also create your own starting with a file of fasta sequences using the makeblastdb command.
* You can extract sequences from a database with the blastdbcmd program

## Running longer jobs with sbatch

The srun command is useful for testing commands and shorter programs. But it has limitations for longer or more intensive jobs.

* Sometimes it takes a long time to get an allocation. With srun you have to wait.
* With srun, you have to wait until your program is done and then remember to log out.

The preferred way to run computation on a cluster like Farm is by submitting jobs that run non-interactively using the command sbatch.

Launching a sbatch job looks like this:

sbatch -p bit150h -t 30 my_script.sh

This will tell the cluster to find a time to allocate resources to my job for 30 minutes, and then run the script my_script.sh

The key to using sbatch is writing a good script. The body of the script is the set of commands you would type in to Bash if you were there to control it. But you start the script with some special commands to tell the cluster how to run your job.

The simplest script looks like this:

#!/bin/bash

module load bio

blastp  \
   -db /group/bit150/blast_databases/refseq_human_protein \
   -query brca1.fasta \
   -out brca1_blast_table.txt \
   -outfmt 7    

* The first line tells the cluster to run this script with the bash shell.
* The second tells it to load the bio module
* Then we give the command for running blast.

Create this script in Notepad++ on a PC (make sure the encoding is set to Unix(LR)) or TextEdit no a Mac (make sure it is set to plain text), call it my_script.sh and move the file to Farm in your Lab_7 directory.

Since this script re-makes the brca1_blast_table.txt file we made earlier, first delete that file so we can see that it will be re-made when we re-run the job:

rm brca1_blast_table.txt

Then submit the job using the sbatch command above. You can check if the job is running by typing: squeue -u username where you replace username with your username. If it's blank, your job is done. Check if the job re-created the brca1_blast_table.txt file correctly.

We can then modify this script to do longer, more intensive jobs:

## Look at the following fasta file:

/group/bit150/Lab_07/mouse_protein_set1.fasta


## Blast with a multi-fasta

If we have many sequences to BLAST, we could save each in a separate file and then write a for loop to iterate over the sequences one-by-one and send each to the blastp program. However, blastp is set up to do this work for us once we've concatenating many sequences together into one file, called a multi-fasta file. In a multi-fasta file, each sequence begins with a header line (denoted by the > character), and then has 1+ lines of sequence. If we submit a file like this to the BLAST programs, they will BLAST each sequence individually against the same database. This obviously takes longer than BLASTing one sequence, so we'll use sbatch:

The following script does a blast using a query file that includes 10 different sequences:

#!/bin/bash

module load bio

blastp  \
   -db /group/bit150/blast_databases/refseq_human_protein \
   -query /group/bit150/Lab_07/mouse_protein_set1.fasta \
   -out mouse_protein_set1_blast_table.txt \
   -outfmt 7    

## Modular scripts
The above script is fine, but it is a bit hard to read, and is also hard to modify. Say you wanted to blast a different gene against a different database? How would you change the script?

The way to write cleaner scripts is to use bash variables. We saw variables earlier when writing for loops. The concept here is the same. We can assign a value to a variable in the top of the script, and then use the variable later:

#!/bin/bash

module load bio

# declare database and query files
database=/group/bit150/blast_databases/refseq_human_protein
query=/group/bit150/Lab_07/mouse_protein_set1.fasta
output_file=mouse_protein_set1_blast_table.txt

blastp  \
   -db $database \
   -query $query \
   -out mouse_protein_set1_blast_table.txt \
   -outfmt 7
Now the call to blastp is much clearer. It's also often helpful to separate directories from file names, and base names from file extensions. A better script might look like this:

#!/bin/bash

module load bio

# declare database directory
database_dir=/group/bit150/blast_databases/

# declare input file directory
query_dir=/group/bit150/Lab_07

# declare output file directory
output_dir=.

# database
database=refseq_human_protein

# query
query=mouse_protein_set1

blastp  \
   -db $database_dir/$database \
   -query $query_dir/${query}.fasta \
   -out $output_dir/${query}_blast_table.txt \
   -outfmt 7


### Notes
When submitting sbatch jobs, two important commands are:

* squeue -p bit150h, or squeue -u bit150-48. These show you if your job is running
* scancel -u bit150-48. This cancels all your running jobs. You can also do: scancel 15968444 to cancel a specific job number
