---
title: "Run your analysis"
teaching: 0
exercises: 40
questions:
- "Can you run POET through apptainer in a condor job?"
- "Can you merge the output files?"
- "Can you export the merged files to your laptop?"
objectives:
- "Test running condor jobs for the CMSSW Open Data container"
- "Learn the `hadd` command for merging ROOT files"
- "Copy files out of TIFR onto your personal machine"
keypoints:
- "The `hadd` command allows you to merge ROOT files that have the same internal structure"
- "Files can be extracted from TIFR to your local machine using `scp`"
- "You can then analyze the POET ROOT files using other techniques from this workshop"
---

## Let's submit a job!

If you have logged out of TIFR, log back in and go to your condor script area:
~~~
$ ssh userXX@ui3.indiacms.res.in
$ cd condorLite/
~~~
{: .language-bash}

One example file list has been created for you. Explore its contents in a text editor or using `cat`:
~~~
$ cat filelists/DYJetsToLL_13TeV_MINIAODSIM.fls
~~~
{: .language-bash}

You will see many ROOT file locations with the ``eospublic" access URL:
~~~
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/004544CB-6DD8-E511-97E4-0026189438F6.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/0047FF1A-70D8-E511-B901-0026189438F4.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/00EB960E-6ED8-E511-9165-0026189438E2.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/025286B9-6FD8-E511-BDA0-0CC47A78A418.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/02967670-70D8-E511-AAFC-0CC47A78A478.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/02AB0BD2-6ED8-E511-835B-00261894393A.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/02FE246D-71D8-E511-971A-0CC47A4D761A.root
root://eospublic.cern.ch//eos/opendata/cms/mc/RunIIFall15MiniAODv2/DYJetsToLL_M-50_TuneCUETP8M1_13TeV-madgraphMLM-pythia8/MINIAODSIM/PU25nsData2015v1_76X_mcRun2_asymptotic_v12_ext1-v1/10000/0626FEB3-70D8-E511-A5B1-0CC47A4D765A.root
...and more...
~~~
{: .output}

Submit condor jobs that will each process the first 5000 events from a list of 2 MiniAOD files. Since this is a small-scale test, cap the number of jobs at 4 (this will not process the entire Drell-Yan dataset). You have two options for submitting these jobs. First, you could reference the file list:
~~~
$ python3 scripts/makeCondorJobs.py  -f filelists/DYJetsToLL_13TeV_MINIAODSIM.fls --tag DYJetsToLL_v1 -n 2 -j 4 -e 5000 --run_template templates/runScript.tpl.sh -s
~~~
{: .language-bash}

Alternately, you can reference the recid of this Drell-Yan dataset:
~~~
$ python3 scripts/makeCondorJobs.py --recid 16446 --tag DYJetsToLL_v1 -n 2 -j 4 -e 5000 --run_template templates/runScript.tpl.sh -s

~~~
{: .language-bash}

You will first be asked to confirm that you really want to submit jobs, because we have included the `-s` argument in this command. Type `y` to confirm.
If you wish to inspect the submission scripts first, leave off `-s` and use the `condor_submit` command printed in the output to submit the jobs later.
You will see something like the following output when you submit jobs, with slight differences between the filelist `-f` and `--recid` submission options:
~~~
[userXX@ui3 condorLite]$ python3 scripts/makeCondorJobs.py  -f filelists/DYJetsToLL_13TeV_MINIAODSIM.fls --tag DYJetsToLL_v1 -n 2 -j 4 -e 5000 --run_template templates/runScript.tpl.sh -s 
Do you really want to submit the jobs to condor pool ? y
 Number of jobs to be made  4
 Number of events to process per job   5000
 Tag for the job  DYJetsToLL_v1
 Output files will be stored at  /home/userXX/condorLite/results/odw_poet/poetV1_DYJetsToLL_v1/
 File list to process :  filelists/DYJetsToLL_13TeV_MINIAODSIM.fls

Making Jobs in templates/runScript.tpl.sh for files from filelists/DYJetsToLL_13TeV_MINIAODSIM.fls

4 Jobs made !
         submit file  : /home/userXX/condorLite/Condor/odw_poet/poetV1_DYJetsToLL_v1//jobpoetV1_DYJetsToLL_v1.sub


Condor Jobs can now be submitted by executing :
condor_submit /home/userXX/condorLite/Condor/odw_poet/poetV1_DYJetsToLL_v1//jobpoetV1_DYJetsToLL_v1.sub
Submitting job(s)....
4 job(s) submitted to cluster CLUSTERID.  # your CLUSTERID will be a number
~~~
{: .output}

## Monitoring condor jobs

HTCondor supports many commands that can provide information on the status of job queues and a user's submitted jobs. Details are availabel in the HTCondor manual for [managing a job](https://htcondor.readthedocs.io/en/latest/users-manual/managing-a-job.html). Three extremely useful commands are shared here.

To see the status of your jobs:
~~~
$ condor_q
~~~
{: .language-bash}

~~~
-- Schedd: ui3.indiacms.res.in : <144.16.111.98:9618?... @ 01/03/24 10:06:52
OWNER  BATCH_NAME      SUBMITTED   DONE    RUN    IDLE  TOTAL JOB_IDS
userXX ID: CLUSTERID  1/3  10:03      _      5      _      5  CLUSTERID.JOBIDs

Total for query: 5 jobs; 0 completed, 0 removed, 0 idle, 5 running, 0 held, 0 suspended 
Total for userXX: 5 jobs; 0 completed, 0 removed, 0 idle, 5 running, 0 held, 0 suspended 
Total for all users: 5 jobs; 0 completed, 0 removed, 0 idle, 5 running, 0 held, 0 suspended
~~~
{: .output}

This command shows the cluster identification number for each set of jobs, the submission time, how many jobs are running or idle, and the individual id numbers of the jobs. 

To remove a job cluster that you would like to kill:
~~~
$ condor_rm CLUSTERID   # use CLUSTERID.JOBID to kill a single job in the cluster
~~~
{: .language-bash}

~~~
All jobs in cluster CLUSTERID have been marked for removal
~~~
{: .output}


## Job output

These short test jobs will likely only take a few minutes to complete. The submission command output points out the directories that will contain the
condor job information and the eventual output files from the job. You can study the condor job's executable script, resource use log, 
error file, and output file in the newly created `Condor` folder:

~~~
[userXX@ui3 condorLite]$ ls Condor/odw_poet/poetV1_DYJetsToLL_v1/ # each unique "tag" you provide when submitting jobs will get a unique folder
Job_1  Job_2  Job_3  Job_4  jobpoetV1_DYJetsToLL_v1.sub 

[userXX@ui3 condorLite]$ ls Condor/odw_poet/poetV1_DYJetsToLL_v1/Job_1
poetV1_DYJetsToLL_v1_1_run.sh  run.1195980.log  run.1195980.stderr  run.1195980.stdout
~~~
{: .language-bash}

The output files can be found in the `results` folder:

~~~
[userXX@ui3 condorLite]$ ls -lh results/odw_poet/poetV1_DYJetsToLL_v1/
total 32M
-rw-r--r-- 1 user1 user1 7.8M Jan  3 21:47 outfile_1_DYJetsToLL_v1_numEvent5000.root
-rw-r--r-- 1 user1 user1 7.8M Jan  3 21:46 outfile_2_DYJetsToLL_v1_numEvent5000.root
-rw-r--r-- 1 user1 user1 7.9M Jan  3 21:46 outfile_3_DYJetsToLL_v1_numEvent5000.root
-rw-r--r-- 1 user1 user1 7.8M Jan  3 21:47 outfile_4_DYJetsToLL_v1_numEvent5000.root
~~~
{: .language-bash}

We can take advantage of the fact that the TIFR cluster also has ROOT installed by default to inspect one of these output files:
~~~
[userXX@ui3 condorLite]$ root results/odw_poet/poetV1_DYJetsToLL_v1/outfile_1_DYJetsToLL_v1_numEvent5000.root

   ------------------------------------------------------------------
  | Welcome to ROOT 6.24/08                        https://root.cern |
  | (c) 1995-2021, The ROOT Team; conception: R. Brun, F. Rademakers |
  | Built for linuxx8664gcc on Sep 29 2022, 13:04:57                 |
  | From tags/v6-24-08@v6-24-08                                      |
  | With c++ (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44)                 |
  | Try '.help', '.demo', '.license', '.credits', '.quit'/'.q'       |
   ------------------------------------------------------------------

root [0] 
Attaching file results/odw_poet/poetV1_DYJetsToLL_v1/outfile_1_DYJetsToLL_v1_numEvent5000.root as _file0...
(TFile *) 0x2b35240
root [1] _file0->ls()  # list the contents of the TFile object
TFile**         results/odw_poet/poetV1_DYJetsToLL_v1/outfile_1_DYJetsToLL_v1_numEvent5000.root
 TFile*         results/odw_poet/poetV1_DYJetsToLL_v1/outfile_1_DYJetsToLL_v1_numEvent5000.root
  KEY: TDirectoryFile   myelectrons;1   myelectrons
  KEY: TDirectoryFile   mymuons;1       mymuons
  KEY: TDirectoryFile   mytaus;1        mytaus
  KEY: TDirectoryFile   myphotons;1     myphotons
  KEY: TDirectoryFile   mypvertex;1     mypvertex
  KEY: TDirectoryFile   mygenparticle;1 mygenparticle
  KEY: TDirectoryFile   myjets;1        myjets
  KEY: TDirectoryFile   myfatjets;1     myfatjets
  KEY: TDirectoryFile   mymets;1        mymets
~~~
{: .language-bash}

Each of these `TDirectoryFile` objects are directories that contain a tree called `Events`. 
For any of the folders, we can access the tree on the command line for quick tests by 
creating a `TTree` object:

~~~
root [4] TTree *tree = (TTree*)_file0->Get("myelectrons/Events");

root [5] tree->GetEntries() # Confirm the number of events in the tree
(long long) 5000

root [6] tree->Print()  # Print out the list of branches available
******************************************************************************
*Tree    :Events    : Events                                                 *
*Entries :     5000 : Total =         2096486 bytes  File  Size =     893419 *
*        :          : Tree compression factor =   2.34                       *
******************************************************************************
*Br    0 :numberelectron : Int_t number of electrons                         *
*Entries :     5000 : Total  Size=      20607 bytes  File Size  =       3509 *
*Baskets :        1 : Basket Size=      32000 bytes  Compression=   5.72     *
*............................................................................*
*Br    1 :electron_e : vector<float> electron energy                         *
*Entries :     5000 : Total  Size=     100130 bytes  File Size  =      49609 *
*Baskets :        4 : Basket Size=      32000 bytes  Compression=   2.01     *
*............................................................................*
*Br    2 :electron_pt : vector<float> electron transverse momentum           *
*Entries :     5000 : Total  Size=     100150 bytes  File Size  =      49382 *
*Baskets :        4 : Basket Size=      32000 bytes  Compression=   2.02     *
*............................................................................*

... and more ...
~~~
{: .language-bash}



## Merge output files

**FIXME: here let's show them where the files will be and how to use hadd to merge them. 
HERE'S THE OPPORTUNITY TO SHOW APPTAINER IN THE COMMAND LINE -- can we use the ROOT container
with apptainer exec to do the hadd? I've also just referenced the native ROOT on this cluster in the section above. 
That's fine -- we could make any apptainer command version be a "callout" labeled "what if my cluster doesn't have ROOT?"**
The output files from codor jobs can be large in number, and we might want to club multiple output root files into a single file.A sample use case will be an instance when we want to merge the POET ouputs files from a specific opendata-datset to a single file. We can use `hadd` tool to achive this

> ## What if my cluster doesn’t have ROOT
> If your cluster does-not have a local instalation of `hadd` (`hadd` is a pakage that comes along with the `root` instalation) , you can use the docker contaner for `root`.To acces the `hadd` from the container, we launch a contaner instance interactively.
~~~
$ apptainer shell --bind results/:/results   docker://gitlab-registry.cern.ch/cms-cloud/root-vnc:latest
~~~
{: .language-bash}
> Here we mount the `result` folder as `/results` folder inside the container. Now we are ready to use hadd availabe in the container.
~~~
Apptainer $ hadd /results/DYJetsToLL_v1.root /results/odw_poet/poetV1_DYJetsToLL_v1/*.root
~~~
{: .language-bash}

~~~~
$ hadd DYJetsToLL_v1.root results/odw_poet/poetV1_DYJetsToLL_v1/*.root
~~~
{: .language-bash}
~~~
hadd Target file: DYJetsToLL_v1.root
hadd compression setting for all output: 1
hadd Source file 4: results/odw_poet/poetV1_DYJetsToLL_v1/outfile_1_DYJetsToLL_v1_numEvent5000.root
hadd Source file 5: results/odw_poet/poetV1_DYJetsToLL_v1/outfile_2_DYJetsToLL_v1_numEvent5000.root
hadd Source file 6: results/odw_poet/poetV1_DYJetsToLL_v1/outfile_3_DYJetsToLL_v1_numEvent5000.root
hadd Source file 8: results/odw_poet/poetV1_DYJetsToLL_v1/outfile_5_DYJetsToLL_v1_numEvent5000.root
hadd Target path: DYJetsToLL_v1.root:/
hadd Target path: DYJetsToLL_v1.root:/myelectrons
hadd Target path: DYJetsToLL_v1.root:/mymuons
hadd Target path: DYJetsToLL_v1.root:/mytaus
hadd Target path: DYJetsToLL_v1.root:/myphotons
hadd Target path: DYJetsToLL_v1.root:/mypvertex
hadd Target path: DYJetsToLL_v1.root:/mygenparticle
hadd Target path: DYJetsToLL_v1.root:/myjets
hadd Target path: DYJetsToLL_v1.root:/myfatjets
hadd Target path: DYJetsToLL_v1.root:/mymets
~~~
{: .output}
This commad will produce a root file, DYJetsToLL_v1.root, merging the trees available inside all the files matching `results/odw_poet/poetV1_DYJetsToLL_v1/*.root`

~~
{: .callout}

> ## My output files are large
> POET output files can be easily be many MB, scaling with the number of events processed. 
> If your files are very large, merging them is not recommended -- it requires additional
> storage space in your account (until the unmerged files can be deleted) and can make 
> transfering the files out very slow.
{: .callout}

## Copy output files out of TIFR

As we've shown in earlier lessons, analysis of the POET ROOT files can be done with ROOT or Python tools, typically on your local machine. 
To extract files from TIFR for local analysis, use the `scp` command: 

~~~
$ scp -r userXX@ui3.indiacms.res.in:/home/userXX/condorLite/results/odw_poet/poetV1_DYJetsToLL_v1/ .
~~~
{: .language-bash}

~~~
user1@ui3.indiacms.res.in's password: 
outfile_2_DYJetsToLL_v1_numEvent5000.root     100% 7977KB   2.5MB/s   00:03    
outfile_3_DYJetsToLL_v1_numEvent5000.root     100% 7997KB   5.3MB/s   00:01    
outfile_1_DYJetsToLL_v1_numEvent5000.root     100% 7979KB   5.8MB/s   00:01    
outfile_4_DYJetsToLL_v1_numEvent5000.root     100% 7968KB   6.0MB/s   00:01    
~~~
{: .output}

You can also transfer individual files, such as a single merged file per dataset. **Now you're ready to dive in to your physics analysis!**

{% include links.md %}

