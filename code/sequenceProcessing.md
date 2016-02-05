# Bacterial Microbiome and Obesity Meta-analysis Sequence Processing Pipeline
Marc Sze  
January 26, 2016  
#### **Preamble**  
Each study is brought up one at a time.  This recount starts from time of download all the way to generation of the final shared file used for the meta-analysis in the study.  You will also need the latest version of mothur, SRA Toolkit, Metaphlan2, and Python 2.7 installed on your system.  There are also a few reference files that you will need.  The first is the [SILVA reference](http://www.mothur.org/wiki/Silva_reference_files) database and the second is the [RDP database](http://www.mothur.org/wiki/RDP_reference_files) of which we used release 102 and version 9 respectively.  Finally, any entered code is denoted will be surronded with ''.  As an example, ''print("Hello World)''.
<br>
<br>

##### **Baxter et al. 2016:**
This data set consisted of controls without colorectal cancer (CRC) and those with CRC.  The sequence reads of only the controls who did not have CRC were obtained directly from the first author since the data was generated within the Schloss lab.  However, the sequences can be found on the SRA under the accession number SRP062005. Only the raw 16S sequence reads for normal controls were processed  from the Sequence Read Archive (SRA) and subsequently used in the analysis.  So here we go....
<br>
<br>
I'm sure a smarter person could seperate the specific files that are normal only but I'm lazy and we had the storage space so I downloaded the entire data set.  
<br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/SRP/SRP062/SRP062005/''
<br>
''fastq-dump \*.sra''
<br><br>
The metadata can be found and downloaded by searching SRP062005 at the following site: http://www.ncbi.nlm.nih.gov/Traces/study/
<br><br>
The simple python program (Python 3.0) used to generate the .file file needed to make contigs can be found in this repository titled: *MSze.makefile.good.trial2.py* and the resulting file is titled *mothur.data.file*.  All you need to make this work is to have only the necessary fastq files in the respective folder.
<br><br>
Before getting started enter mothur and curate the silva.fasta for the v4 region with the following commands:
<br><br>
''pcr.seqs(fasta=silva.bacteria.fasta, start=11894, end=25319, keepdots=F)''
<br>
''system(mv silva.bacteria.pcr.fasta silva.v4.fasta)''
<br><br>

Next we utilized the batch mode process and can come back the next day to see everything completed.  Our command was the following:
<br><br>
''/share/scratch/schloss/mothur/mothur data1.batch''
<br>
<br>
If you thought this was difficult to get data just wait until later on in this pipeline to see just how crazy it can get trying to get the data to a useable point.
<br>
<br>

##### **Ross et al. 2015:**
This data set consisted of normal individuals from the Cameron Count Hispanic Cohort (CCHC).  The paper for this data set can be found [here](http://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-015-0072-y).  The project accession number in the SRA is SRP053023.  We can download the entire data set again with the code mentioned above but with one small substitution at the ending.  Here we go......
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/SRP/SRP053/SRP053023/''
<br>
<br>
Now since there was no sff file provided in this data set even though it is a 454 data set (I found this out when I tried to run the command -> sff-dump \*.sra).  We have to use the fastq function of the SRA Tool kit.
<br>
<br>
''fastq-dump \*.sra''
<br>
<br>
Because of this difference there are a few changes in this processing versus the standardized mothur 454 protocol.  First, instead of the sff.info() command we have to use the fastq.info() command.  Second, and perhaps more importantly, because we don't have the sff file we have to use the quality scores.  We used the author specified defaults of *qwindowaverage=25* and *qwindowsize=50*.  It should also be noted that we had *maxambig=5* and *maxhomop=8*.  Since the data come seperated already we do not need an oligo file and all we have to do is loop the mothur *trim.seqs* command. You can run a bash or shell program to get all of this done with either *#!/bin/sh* or *#!/bin/bash*.  So in a text editor:    

''#!/bin/sh  
for sample in \$(ls \*sra)  
do  
fastq-dump \$sample  
done
<br><br>
for sample2 in \$(ls \*fastq)  
do  
/share/scratch/schloss/mothur/mothur "#fastq.info(fastq=\$sample2)"  
rm \*logfile  
done
<br><br>
for TRIM in \$(ls ERR\*.fasta)  
do  
QFILE=\${TRIM//fasta/qual}  
/share/scratch/schloss/mothur/mothur "#trim.seqs(fasta=\$TRIM, qfile=\$QFILE, maxambig=5, maxhomop=8, qwindowaverage=25, qwindowsize=50, processors=8)"  
done  
rm \*.fastq''
<br><br>
The next lines of code are used to combine the fasta file, create a names file with the mothur unique command, and create a combined group file for subsequent processing.
<br><br>
''cat \*trim.fasta > combined.trim.fasta  
/share/scratch/schloss/mothur/mothur "#unique.seqs(fasta=combined.trim.fasta)"  
grep -e '^>' combined.trim.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed 's/\\./\_/g' test2.fasta > test3.fasta  
sed 's/_.\*//' test3.fasta > test4.fasta  
paste -d" " test2.fasta test4.fasta > combined.groups  
exit 0''
<br><br>
After this little program is run we follow the mothur 454 SOP as written and can use the following command to complete the processing using the put together *Ross.batch* file and running mothur.  One final difference from the standardized mothur approach that was made was during the screen.seqs command after alignment a hard cutoff *minlength=200* and *maxlength=250* was used.
<br><br>
''/share/scratch/schloss/mothur/mothur Ross.batch''
<br><br>
That completes the processing for this data set.  Simple so far I hope.  On to the next data set.
<br><br>

##### **Goodrich et al. 2014:**  
This is a second larger twin data set that builds on the Turnbaugh et al. findings.  The paper can be found [here](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4255478/).  The individuals that participated in the study were part of the TwinsUK registry and is somewhat heavily skewed toward women.  The raw sequence data can be found on the SRA under ERP006339.  So using wget we can download the data with the following code:
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/ERP/ERP006/ERP006339/''
<br><br>
 I started off with analyzing the data separately while sticking the the default mothur SOP for MiSeq processing.....
<br><br>
''#!/bin/sh  
\# Unpack sra files  
\# mkdir sra.complete  
for start in \$(ls \*sra)  
do  
fastq-dump \$start  
mv \$start sra.complete  
done
<br><br>
\# Unpack fastq files  
mkdir fastq.complete  
mkdir logfile.complete  
for info in \$(ls \*fastq)  
do  
/share/scratch/schloss/mothur/mothur "#fastq.info(fastq=\$info)"  
mv \$info fastq.complete  
mv \*.logfile logfile.complete  
done
<br><br>
\# Screen the fasta file that came from the fastq  
mkdir fasta.complete  
for next in \$(ls \*fasta)  
do  
/share/scratch/schloss/mothur/mothur "#screen.seqs(fasta=\$next, maxambig=0, maxlength=275, processors=8)"  
mv \$next fasta.complete \# should store silva file in here and then move it working directory  
mv \*.logfile logfile.complete  
done
<br><br>
mv ~/twinsUK/fasta.complete/silva.v4.fasta ~/twinsUK  
\# Align the screened fasta file to V4 region of SILVA  
mkdir good.fasta.complete  
for sample2 in \$(ls \*good.fasta)  
do  
/share/scratch/schloss/mothur/mothur "#align.seqs(fasta=\$sample2, reference=silva.v4.fasta, processors=8)"  
mv \$sample2 good.fasta.complete  
mv \*logfile logfile.complete  
done
<br><br>
\#Screen the aligned sequences to specific location  
mkdir align.complete  
for sample3 in \$(ls \*.align)  
do  
/share/scratch/schloss/mothur/mothur "#screen.seqs(fasta=\$sample3, start=1968, end=11546, maxhomop=8, processors=8)"  
mv \$sample3 align.complete  
done
<br><br>
\# Filter the screened sequences to get a fasta file back  
mkdir align.good.complete  
for sample4 in \$(ls \*good.align)  
do  
/share/scratch/schloss/mothur/mothur "#filter.seqs(fasta=\$sample4, vertical=T, trump=., processors=8)"  
mv \$sample4 align.good.complete  
mv \*logfile logfile.complete  
done
<br><br>
\# Make the combined fasta file  
mkdir filter.fasta.complete  
cat \*filter.fasta > combined.fasta  
mv \*filter.fasta filter.fasta.complete
<br><br>
\# Make the combined group file  
grep -e '^>' combined.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed 's/\\..\*\\$//' test2.fasta > test3.fasta  
paste -d"\\t" test2.fasta test3.fasta > combined.groups''
<br><br>
After this program was run I thought perhaps naively that I could simply run a set a batch file and be done with this data set.  Well not quite.  The sheer size and the way our server was configured meant that if I called much more than 130GB of RAM my job would run the risk of starting over regularly. So I did use the batch method but only upto *pre.cluster*.  After this the *chimera.uchime* step was just taking too long so I seperated everything again and did the *chimera.uchime* separately.  This also meant having to redo the unique seqs step.  The code for this is below:
<br><br>
''/share/scratch/schloss/mothur/mothur Goodrich.batch''
<br><br>
So now the biggest challenge that I have found so far is getting the data into a distance matrix not based on the closed-reference approach.  The file is just too big for cluster.split so I'm going to have to find an alternative way.

<br><br>

##### **Escobar et al. 2014:**
This data set consisted of normal individuals from Medellin, Colombia South America.  The paper for this data set can be found [here](http://bmcmicrobiol.biomedcentral.com/articles/10.1186/s12866-014-0311-6).  The project accession number in the ENA is ERP003466.  We can download the entire data set again with the code mentioned above but with a few substitutions versus the previous two studies.  Here we go......
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/ERP/ERP003/ERP003466/''
<br>
<br>
Now just like the previous data set there was no sff file provided in this data set even though it is a 454 data set (I found this out, again, when I tried to run the command -> sff-dump \*.sra).  We have to use the fastq function of the SRA Tool kit.
<br>
<br>
''fastq-dump \*.sra''
<br>
<br>
Because of this difference there are a few changes in this processing versus the standardized mothur 454 protocol.  First, instead of the sff.info() command we have to use the fastq.info() command.  Second, and perhaps more importantly, because we don't have the sff file we have to use the quality scores.  Unlike the Ross, et al study we used the mothur specified defaults of *qwindowaverage=35* and *qwindowsize=50*.  It should also be noted that we had *maxambig=0* and *maxhomop=8*.  Since the data come seperated already we do not need an oligo file and all we have to do is loop the mothur *trim.seqs* command. You can run a bash or shell program to get all of this done with either *#!/bin/sh* or *#!/bin/bash*.  So in a text editor:    

''#!/bin/sh  
for sample in \$(ls \*sra)  
do  
fastq-dump \$sample  
done
<br><br>
for sample2 in \$(ls \*fastq)  
do  
/share/scratch/schloss/mothur/mothur "#fastq.info(fastq=\$sample2)"  
rm \*logfile  
done
<br><br>
for TRIM in \$(ls SRR\*.fasta)  
do  
QFILE=\${TRIM//fasta/qual}  
/share/scratch/schloss/mothur/mothur "#trim.seqs(fasta=\$TRIM, qfile=\$QFILE, maxambig=0, maxhomop=8, qwindowaverage=35, qwindowsize=50, processors=8)"  
done  
rm \*.fastq''
<br><br>
The next lines of code are used to combine the fasta file, create a names file with the mothur unique command, and create a combined group file for subsequent processing.
<br><br>
''cat \*trim.fasta > combined.trim.fasta  
/share/scratch/schloss/mothur/mothur "#unique.seqs(fasta=combined.trim.fasta)"  
grep -e '^>' combined.trim.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed 's/\\./\_/g' test2.fasta > test3.fasta  
sed 's/_.\*//' test3.fasta > test4.fasta  
paste -d" " test2.fasta test4.fasta > combined.groups  
exit 0''
<br><br>
After this little program is run we follow the mothur 454 SOP as written and can use the following command to complete the processing using the put together *Escobar.batch* file and running mothur.  Again like the Ross et al data set one final difference from the standardized mothur approach that was made was during the screen.seqs command after alignment a hard cutoff *minlength=200* and *maxlength=250* was used.
<br><br>
''/share/scratch/schloss/mothur/mothur Escobar.batch''
<br><br>
That completes the processing for this data set.  So far so good, no major problems.  On to the next data set.
<br><br>

##### **Zupancic et al. 2012:**
This data set consisted of normal individuals from an Amish community living in Lancaster County, Pennsylvania.  The paper for this data set can be found [here](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.00430526).  The project accession number in the SRA is SRP002465.  We can download the entire data set again with the code mentioned above but with a few substitutions.  Here we go......
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/SRP/SRP002/SRP002465/''
<br><br>
Now this data set included the sff files so we can use sffinfo function and start from the very start of the mothur 454 pipleine.  Plus one for this data set so far.  The below program was used and you can use either *#!/bin/sh* or *#!/bin/bash* and text editor to create it and then run it.  Also you should be aware that this part of the processing will take a LONG time to get through.
<br><br>
''#!/bin/sh
for sample in \$(ls \*sra)  
do  
sff-dump \$sample  
done
<br><br>
for sample2 in \$(ls \*sff)  
do  
/share/scratch/schloss/mothur/mothur "#sffinfo(sff=\$sample2, flow=T)"  
rm \*logfile  
done
<br><br>
for FLOW in \$(ls \*flow)  
do  
/share/scratch/schloss/mothur/mothur "#trim.flows(flow=\$FLOW, processors=8)"  
rm \*logfile  
done
<br><br>
for SHHH in \$(ls \*files)  
do  
/share/scratch/schloss/mothur/mothur "#shhh.flows(file=\$SHHH, processors=8)"  
rm \*logfile  
done
<br><br>
for TRIMS in \$(ls \*shhh.fasta)  
do  
NAMES=\${TRIMS//fasta/names}  
/share/scratch/schloss/mothur/mothur "#trim.seqs(fasta=\$TRIMS, name=\$NAMES, maxhomop=8, maxambig=0, minlength=200, processors=8)"  
rm \*logfile  
done  
exit 0''
<br><br>
Now, I had a few extra lines that combined the finalized files together but when I went to analyze these they gave some weird errors.  So I needed to make a second program to sort through these errors and analyze the data separately still.  One thing I noticed was that by using the standard workflow a large number of files ended up with 0 or very few sequences passing the quality filters.  The other problem which I delayed dealing with in this data set was that many samples were done sequenced more than once. This in itself is not a problem but the sequence ID associated with these sequences from different files were the exact same and so were carbon copies.  This wreaked havoc when I tried to combined the data together.  But I'll deal with that problem later.  I'm just going to continue on with the pipeline with the file seperated.  I will however, have to deal with this before the preclustering step.
<br><br>
So here is the second program used to keep processing the sequences separately rather than as one combined fasta file.....
<br><br>
''#!/bin/sh  
\#Align the fasta file to the SILVA database and create directories to move files into    
mkdir good.fasta.complete  
mkdir reference.files  
mkdir logifle.complete  
mv silva* reference files  
mv trainset9* reference files  
for sample2 in \$(ls \*.fasta)  
do  
/share/scratch/schloss/mothur/mothur "#align.seqs(fasta=\$sample2, reference=reference.files/silva.bacteria.fasta, flip=t, processors=8)"  
mv \$sample2 good.fasta.complete  
mv \*logfile logfile.complete  
done
<br><br>
\#Screen the align sequences  
mkdir align.complete  
for sample3 in \$(ls \*.align)  
do  
/share/scratch/schloss/mothur/mothur "#screen.seqs(fasta=\$sample3, end=6453, optimize=start, criteria=95, processors=8)"  
mv \$sample3 align.complete
mv \*logfile logfile.complete  
done
<br><br>
\#filter the screen sequences  
mkdir align.good.complete  
for sample4 in \$(ls \*good.align)  
do  
/share/scratch/schloss/mothur/mothur "#filter.seqs(fasta=\$sample4, vertical=T, trump=., processors=8)"  
mv \$sample4 align.good.complete  
mv \*logfile logfile.complete  
done  
exit 0''
<br><br>
Right, now I need to pre cluster the data and I can't get away with analyzing each file separately anymore.  However, remember from up above there is something wierd going on in that the unique sequence ID are duplicated for many samples and there were a whole bunch of samples that did not survive quality filtering.  Since some samples had two samples others had 6 samples to be fair I decided to take the first sample from an individual that had sequences in them (so greater than 0 MB).  Now, this was done when I was still truly a novice in using shell scripting so I did this the "hard" way (i.e. the following method is pretty manual with very little automation so someone can make this more automated to improve on this process).  Essentially if a sample had sequence in it the indivdiual study ID was assigned to the fasta file and stored in a new directory.  Those that either had 0 MB after this we carried over and the next sequence ID file was used.  This was done until all samples had an indivdiual ID.  In my novice approach the only thing I automated was the original copying of the sequence ID file to a individual study ID file.  I then visually checked the size and manually sorted the files into directories.  From this I created a new list to run through.  The lists that were used were *combined2*, *combined3*, *combined4*, *combined5*, *combined6*, *combined7*, and *combined8* and can be found in this same repository.
<br><br>
As an aside I have learned a lot about coding doing this project and if I were to do this analysis again for this study I would automate this by first generated the list of sequence IDs for each respective Indivdiual and then adding an argument that if the file size is larger than 0 MB to stop and move it.  If it was 0 MB the program would move on to the next sequence ID file for that Indivdiual.
<br><br>
Now that this little problem is out of the way and I have been perfectly upfront with how I dealt with this lets move on to the final part of the processing pipeline for this data set.
<br><br>
For this step I took the individual sample ID named fasta files and created a list which I looped with a shell script to create a group file which can then be combined along with the fasta file.  The code I used for this was:
<br><br>
''#!/bin/sh  
mkdir trim.fasta.complete2  
\#Make loop to create a group file  
groupN="106 107 108 113 116 11 126 128 12 13 143 14 150 156 157 160 162 163 169 172 177 181 18 190 191 192 19 1 201 202 212 214 216 217 219 225 226 230 242 24 269 26 275 283 287 289 291 292 297 299 29 2 313 323 327 333 33 346 347 349 351 352 354 358 359 361 367 373 376 388 38 392 397 398 401 406 410 411 415 416 421 423 424 427 429 444 44 453 455 459 463 464 467 46 471 473 474 475 476 481 482 483 484 488 48 49 50 51 523 52 53 54 59 66 68 6 74 77 81 84 87 92 99 p0 p105 p108 p111 p112 p113 p114 p115 p116 p117 p118 p119 p120 p121 p122 p123 p124 p125 p126 p127 p130 p132 p133 p134 p135 p136 p137 p138 p139 p13 p140 p142 p145 p148 p14 p150 p151 p152 p153 p154 p155 p156 p158 p159 p15 p160 p16 p17 p22 p23 p24 p25 p26 p33 p34 p36 p37 p38 p3 p41 p42 p43 p44 p46 p49 p4 p57 p58 p63 p68 p69 p71 p72 p73 p74 p77 p79 p7 p81 p82 p85 p86 p87 p8 p90 p91 p95 p98"  
for V in \${groupN}  
do  
grep -e '^>' \${V}.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed "s/$/\\t\$V/" test2.fasta > \${V}.groups  
mv \${V}.fasta trim.fasta.complete2  
rm test.fasta  
rm test2.fasta  
done
<br><br>
cd trim.fasta.complete2  
cat \*fasta > combined.fasta  
cat \*groups > combined.groups  
/share/scratch/schloss/mothur/mothur "#unique.seqs(fasta=combined.fasta)"''
<br><br>
Now that this has been completed we can create a batch file for the remaining commands and analyze the data altogether.  One final comment is that for the *screen.seq* command I did not set a *maxhomop=8*.  Due to the large loss (which you will see from the subsampling) from this data set I did not want to be picky.
<br><br>
''/share/scratch/schloss/mothur/mothur Zupancic.batch''
<br><br>
With that you should now be able to generate the subsample shared file that was used for the downstream analysis.  This was a headache of a data set to work with.  I think the next one will be better. At least I hope so....
<br><br>


##### **Yatsunenko et al. 2012:**  
This data set was another famous one looking at the microbiome of indivdiuals at different geographical locations and pretty different diets.  The link to the paper is [here](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3376388/).  A couple problems from a data processing and sequence clean up point of view is that they used a HiSeq, single end, 100bp total length, sequencing run.  This wreaked absolute havoc with a lot of the downstream steps mostly with trying to figure out a way to reduce the artificial diversity within the files.  Oh... and the data was not on a public database but stored on MG-RAST.  I had a lot trouble downloading the data on command line all at once (maybe I'm just not smart enough to do it).  So I had to manually download the sequence files.Thats right I am just a glutton for punishment.    After you have downloaded the files the first part of the code is:
<br><br>
''#!/bin/sh  
mkdir fna.done  
mkdir logfiles.done  
mkdir bad.accnos.done
<br><br>
mkdir good.fna.done  
for sample in \$(ls \*.fna)  
do  
/share/scratch/schloss/mothur/mothur "#align.seqs(fasta=\$sample, reference=silva.bacteria.fasta, processors=8)"  
mv \$sample good.fna.done  
mv \*logfile logfiles.done  
done''
<br><br>
For the next code it might be best to just double check that the start and end are what is written here.  To make it go faster I made a custom pcr.seqs based on the start and end of the sequences on the aligned file.  It shouldn't make a difference but.....you never know.
<br><br>
''#!/bin/sh  
mkdir align.done  
for sample2 in \$(ls \*.align)  
do  
/share/scratch/schloss/mothur/mothur "#screen.seqs(fasta=\$sample2, start=8, end=2448, maxhomop=8, processors=8)"  
mv \$sample2 align.done  
mv \*logfile logfiles.done  
done
<br><br>
mkdir good.align.done  
for sample3 in \$(ls \*good.align)  
do  
/share/scratch/schloss/mothur/mothur "#filter.seqs(fasta=\$sample3, vertical=T, trump=., processors=8)"  
mv \$sample3 good.align.done  
mv \*logfile logfiles.done  
done
<br><br>
mkdir completed.good.groups  
for sample in \$(ls \*.good.filter.fasta)  
do  
grep -e '^>' \$sample > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed 's/_.\*//' test2.fasta > test3.fasta  
paste -d"\\t" test2.fasta test3.fasta > \${sample}.groups  
rm test.fasta  
rm test2.fasta  
rm test3.fasta  
mv ~/Yatsunenko.Nature/${sample}.groups ~/Yatsunenko.Nature/completed.good.groups   
done
<br><br>
From here I thought I might be able to get away with combining the data set and running everything together.  Well that was not the case since it is just too large for the memory I have readily available without having the job restarting.  The plan is to seperate everything back out again and run the *chimera.uchime* function.  Then combine them and run the *pre.cluster* function.  The code below shows what was used for this part:
<br><br>
''#!/bin/sh  
cat *fasta > combined.fasta  
cat *groups > combined.groups  
/share/scratch/schloss/mothur/mothur "#unique.seqs(fasta=combined.fasta)"    
/share/scratch/schloss/mothur/mothur Yatsunenko.batch  
done
<br><br>




<br><br>

##### **Human Microbiome Project (HMP) 2011:**  
This data set consisted of healthy individuals that were sampled at multiple body sites and underwent extensive medical work up for metadata collection.  In order to be included in this study you could not have any underlying medical conditions.  The project website can be found [here](http://hmpdacc.org/).  The project accession number in the SRA is SRP002860.  The sequence processing pipeline can be found in an ipython notebook located [here](http://nbviewer.jupyter.org/gist/pschloss/9815766/notebook.ipynb).  The one exception to the workflow provided in the notebook is that you will need to make sure that you do not use a closed reference data base approach for OTU clustering and instead use an open reference approach.
<br><br>

##### **Nam, et al. 2011:**  
This study recruited 20 individuals that were from the area surronding Kyung Hee University in South Korea and can be found [here](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0022109).  The data from this project can be found under the accession number PRJDA6057 and we downloaded the data from DRR000776.  There is some small difference to the wget, which is mostly associated with the directory where the data is stored.  From here you can run the SRA tool kit command to get the sff file.  Once this is done you can then run mothur with the batch file and get the sub sample shared file that was used in this analysis.  We have also provided the oligo file in this repository *korean6.oligos* that was used so that you can easily use it on this data set.
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByRun/sra/DRR/DRR000/DRR000776
<br><br>
sff-dump DRR000776.sra
<br><br>
/share/scratch/schloss/mothur/mothur Nam.batch''
<br><br>
Compared to the previous data sets this was one of the most straight forward to actually process.
<br><br>

##### **Wu, et al. 2011:**  
This study was published in Science using the COMBO data set and can be found [here](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3368382/).  The investigators were mostly interested in investigating dietary patterns its influence of gur microbial enterotypes. The data for this project can be found on the SRA website under the Accession number SRP002424.  So with a little modification we can use wget again to get the data.
<br><br>
''wget -r -np -nd -k ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByStudy/sra/SRP/SRP002/SRP002424/''
<br><br>
From here there was a program that was run to loop through a few of the mothur functions on each individual file.  Since they are already seperated by indivdiual we don't need the oligo file.  So here we go......
<br><br>
''#!/bin/sh  
mkdir sra.complete  
\#run first loop which will extract sff files  
for sample in \$(ls \*sra)  
do  
sff-dump \$sample  
mv \$sample sra.complete
<br><br>
\#create new directories for logfiles and sff's that are complete.  
mkdir logfile.complete  
mkdir sff.complete  
\#Extract info from sff files  
for sample2 in \$(ls \*sff)  
do  
/share/scratch/schloss/mothur/mothur "#sffinfo(sff=\$sample2, flow=T)"  
mv \$sample2 sff.complete  
mv \*logfile logfile.complete  
done
<br><br>
mkdir flow.complete  
\#Trim respective flow files  
for FLOW in $(ls \*flow)  
do  
/share/scratch/schloss/mothur/mothur "#trim.flows(flow=\$FLOW, processors=8)"  
mv $FLOW flow.complete  
mv \*logfile logfile.complete  
done
<br><br>
mkdir file.complete  
\#Make loop to run the shhh.flow and move completed sample to another directory  
for SHHH in \$(ls \*files)  
do  
/share/scratch/schloss/mothur/mothur "#shhh.flows(file=\$SHHH, processors=8)"  
mv \$SHHH file.complete  
mv \*logfile logfile.complete  
done
<br><br>
mkdir shhh.fasta.complete  
for TRIMS in \$(ls \*shhh.fasta)  
do  
NAMES=\${TRIMS//fasta/names}  
/share/scratch/schloss/mothur/mothur "#trim.seqs(fasta=\$TRIMS, name=\$NAMES, maxhomop=8, minlength=200, processors=8)"  
mv \$TRIMS shhh.fasta.complete  
mv \*logfile logfile.complete''  
<br><br>
So for the next part I needed to make a group file so I renamed the fasta file to represent the sampleID label so that I could with more ease create the labels I needed within the group file. The code for it is....
<br><br>
''samples="SRR327637 SRR327638 SRR327639 SRR327640 SRR327641 SRR327642 SRR327643 SRR327644 SRR327645 SRR327646 SRR327647 SRR327648 SRR327649 SRR327650 SRR327651 SRR327652 SRR327653 SRR327654 SRR327655 SRR327656 SRR327657 SRR327658 SRR327659 SRR327660 SRR327661 SRR327662 SRR327663 SRR327664 SRR327665 SRR327666 SRR327667 SRR327668 SRR327669 SRR327670 SRR327671 SRR327672 SRR327673 SRR327674 SRR327675 SRR327676 SRR327677 SRR327678 SRR327679 SRR327680 SRR327681 SRR327682 SRR327683 SRR327684 SRR327685 SRR327686 SRR327687 SRR327688 SRR327689 SRR327690 SRR327691 SRR327692 SRR327693 SRR327694 SRR327695 SRR327696 SRR327697 SRR327698 SRR327699 SRR327700 SRR327701"
<br><br>
for V in \${samples}  
do  
grep -e '^>' \${V}.shhh.trim.good.filter.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta  
sed "s/\$/\\t\$V/" test2.fasta > \${V}.groups  
mv \${V}.shhh.trim.good.filter.fasta filter.fasta.complete  
rm test.fasta  
rm test2.fasta  
echo \$V  
done''
<br><br>
From here you can go to the individual folders and find the final version of the respective fasta, names, and groups files for every indivdiual and combine them into a single larger file.  In essence you don't need the names file since mothur will re-generate one when you run the *unique.seqs* command.  This was the route that I opted for since it meant slightly less typing.  So the only files that I needed to combine are the fasta and groups files.
<br><br>
''cat \*trim.good.filter.fasta > combined.fasta''  
''cat \*groups > combined.groups''
<br><br>
After you have done this you can run a batch file with mothur and that should generate the subsample shared file that we used for this anlaysis.  We lose a large number of samples like in the Zupancic data set since we are subsampling to 1000.  We choose 1000 since it is an arbitrary standard many use but also because we wanted to set a standardized lowest vaule to subsample for that we can hold every data set to.  Anyways by running the final command you should get the files needed to all the downstream analysis.
<br><br>
''/share/scratch/schloss/mothur/mothur Wu.batch''
<br><br>

##### **Arumugam, et al. 2010 (MetaHit):**  
We only used the Danish individuals for this study and left the others alone since many had IBD or other disease conditions.  For this data set Metaphlan2 was used to generate the abundance data.  Metaphlan2 and how well it associates with 16S data is also discussed in the supplemental of the draft manuscript we are writing.  To test this we downloaded ~40 samples from the HMP data set which had both metagenome and 16S data.  We ran the same code that we used with the MetaHit data to generate the abundance data using Metaphlan2 and compared that result with the obtained 16S values.  So here is code that was used below:
<br><br>
''#!/bin/sh  
PATH=\$PATH:~/metaphlan2  
export mpa_dir=~/metaphlan2  
mkdir completed.combined.fq  
test_samples="MH0001 MH0002 MH0003 MH0004 MH0005 MH0006 MH0007 MH0008 
MH0009 MH0010 MH0011 MH0012 MH0013 MH0014 MH0015 MH0016 MH0017 MH0018 
MH0019 MH0020 MH0021 MH0022 MH0023 MH0024 MH0025 MH0026 MH0027 MH0028 
MH0030 MH0031 MH0032 MH0033 MH0034 MH0035 MH0036 MH0037 MH0038 MH0039 
MH0040 MH0041 MH0042 MH0043 MH0044 MH0045 MH0046 MH0047 MH0048 MH0049 
MH0050 MH0051 MH0052 MH0053 MH0054 MH0055 MH0056 MH0057 MH0058 MH0059 
MH0060 MH0061 MH0062 MH0063 MH0064 MH0065 MH0066 MH0067 MH0068 MH0069 
MH0070 MH0071 MH0072 MH0073 MH0074 MH0075 MH0076 MH0077 MH0078 MH0079 
MH0080 MH0081 MH0082 MH0083 MH0084 MH0085 MH0086"
<br><br>
for s in \${test_samples}  
do  
cat \${s}\*.1.fq > \${s}.combined.1.fq  
cat \${s}\*.2.fq > \${s}.combined.2.fq  
mv \${s}\*combined.1.fq completed.combined.fq  
mv \${s}\*combined.2.fq completed.combined.fq  
done''
<br><br>
This code was used to combine the fastq files since there are multiple files per sample.  To actually implement the metaphlan2 program for these files see below:
<br><br>
''for s in \${test_samples}  
do  
metaphlan2.py /mnt/EXT/Schloss-data/msze/test/danish.obesity.metagenome/raw.data.sep/completed.combined.fq/\${s}.combined.1.fq,/mnt/EXT/Schloss-data/msze/test/danish.obesity.metagenome/raw.data.sep/completed.combined.fq/\${s}.combined.2.fq --mpa_pkl db_v20/mpa_v20_m200.pkl --bowtie2db db_v20/mpa_v20_m200 --input_type fastq --bowtie2out profiled_samples/\${s}.bowtie2.bz2 -o profiled_samples/\${s}.txt  
done''
<br><br>
In order to combine all the files into a single data table the following code was run:
<br><br>
''python ~/metaphlan2/utils/merge\_metaphlan\_tables.py profiled\_samples/\*.txt > profiled\_samples/output/merged\_abundance\_table.txt''
<br><br>
For the 16S and metaphlan2 comparison the following code was run to obtain the necessary data tables:
<br><br>
''#!/bin/sh  
PATH=\$PATH:~/metaphlan2  
export mpa_dir=~/metaphlan2
<br><br>
test_samples="SRS012849 SRS012969 SRS013098 SRS013216 SRS013638 SRS013639 SRS016437 SRS016438 SRS016990 SRS017916 SRS020622 SRS021153 SRS021219 SRS022093 SRS024663 SRS043667 SRS045528 SRS045739 SRS049402 SRS049896 SRS050026 SRS053356 SRS053573 SRS053649 SRS054352 SRS055533 SRS056273 SRS058070 SRS063489 SRS063518 SRS064973 SRS074670 SRS074964 SRS075078 SRS075341 SRS075963 SRS076929 SRS077086 SRS077194 SRS077294 SRS077335 SRS077502 SRS077552 SRS077753 SRS077849 SRS078242 SRS078419 SRS078665 SRS097889 SRS098514 SRS098571 SRS098644 SRS098717 SRS098827 SRS0100021 SRS101376 SRS101433 SRS103987 SRS0104197 SRS104311 SRS104400 SRS104485 SRS105153 SRS140492 SRS140513 SRS140645 SRS142503 SRS142505 SRS142599 SRS142712 SRS142890 SRS143070 SRS143181 SRS143342 SRS143417 SRS143598 SRS143780 SRS143876 SRS143991 SRS144362 SRS144506 SRS144537 SRS145497 SRS146764 SRS146812 SRS146813 SRS147022 SRS147088 SRS147139 SRS147346 SRS147445 SRS147652 SRS147766 SRS147919 SRS148196 SRS148424 SRS148721" <br><br>
for s in \${test_samples}  
do  
\#Download and unpack necessary files  
wget -r -np -nd -k ftp://public-ftp.hmpdacc.org/Illumina/PHASEII/stool/${s}.tar.bz2  
tar xvjf \${s}.tar.bz2  
mv \${s}/\${s}.denovo\_duplicates\_marked.trimmed.1.fastq ~/metaphlan2  
mv \${s}/\${s}.denovo\_duplicates\_marked.trimmed.2.fastq ~/metaphlan2  
metaphlan2.py \${s}.denovo\_duplicates\_marked.trimmed.1.fastq,\${s}.denovo\_\duplicates_\marked.trimmed.2.fastq --mpa\_pkl db\_v20/mpa\_v20\_m200.pkl --bowtie2db db\_v20/mpa\_v20\_m200 --input\_type fastq --bowtie2out profiled\_samples/\${s}.bowtie2.bz2 -o profiled\_samples/\${s}.txt  
rm -r \${s}  
rm \${s}.denovo_duplicates_marked.trimmed.1.fastq  
rm \${s}.denovo_duplicates_marked.trimmed.2.fastq  
done''
<br><br>
You can then go into the directory with all the seperate data tables and combined them into one with the following command:
<br><br>
''python ~/metaphlan2/utils/merge\_metaphlan\_tables.py profiled\_samples/\*.txt > profiled\_samples/output/merged\_abundance\_table.txt''
<br><br>
Now I decided, perhaps foolishly, to manually curate the combined file.  What I mean by this is that I manually deleted viruses and eukaryotic cells that were identified instead of using a computer to do this.  I also then manually created data tables with phyla and the lowest taxonomic identification.  The data table with the lowest taxonomic identification information was used to generate alpha diversity measures and to complete and PERMANOVA analysis that was used.  These manually curated files are provided in this repository for you to use as you want.  Finally, I used relative abundance instead of subsampling in these samples.  So I had to redo the relative abundance after removing the viruses and eukaryotic cells.  This part was done in R.
<br><br>

##### **Turnbaugh, et al. 2008:**  
This is the very famous twin study of the microbiome and obesity.  The other two re-analysis articles [@finucane_taxonomic_2014; @walters_meta-analyses_2014] also use this data set.  The raw data is not avialable on the SRA but you can download a fasta file for either the V2 or V6 16S rRNA gene [here](https://gordonlab.wustl.edu/NatureTwins_2008/TurnbaughNature_11_30_08.html).  We used the V2 region.  For the most part it was relatively straight forward, just be aware that this took a lot of RAM to create a distance matrix.  The code I used is below:
<br><br>
''grep -e '^>' combined.trim.fasta > test.fasta  
sed 's/>//' test.fasta > test2.fasta    
sed 's/\\s.\*\$//' test2.fasta > test3.fasta  
paste -d" " test2.fasta test3.fasta > V2.groups''
<br><br>
Once we have a groups file we can just create a batch file and put it into mothur to run and come back when it is completed.
<br><br>
''/share/scratch/schloss/mothur/mothur Turnbaugh.batch''
<br><br>

##### **References:** 
























