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
$ python3 scripts/makeCondorJobs.py  -f filelists/DYJetsToLL_13TeV_MINIAODSIM.fls --tag DYJetsToLL_v1 -n 2 -j 4 -e 5000 --run_template templates/runScript.tpl.sh
~~~
{: .language-bash}

Alternately, you can reference the recid of this Drell-Yan dataset:
~~~
$ python3 scripts/makeCondorJobs.py --recid 16446 --tag DYJetsToLL_v1 -n 2 -j 4 -e 5000 --run_template templates/runScript.tpl.sh

~~~
{: .language-bash}

You will see:
~~~
**FIXME, add output***
~~~
{: .output}

Use the condor monitoring commands shown in the previous episode to monitor your test jobs. 

## Merge output files

**FIXME: here let's show them where the files will be and how to use hadd to merge them. 
HERE'S THE OPPORTUNITY TO SHOW APPTAINER IN THE COMMAND LINE -- can we use the ROOT container
with apptainer exec to do the hadd?**

> ## My output files are large
> POET output files can be easily be many MB, scaling with the number of events processed. 
> If your files are very large, merging them is not recommended -- it requires additional
> storage space in your account (until the unmerged files can be deleted) and can make 
> transfering the files out very slow.
{: .callout}

## Copy output files out of TIFR

**FIXME: show scp (and scp -r) on a file (or folder of files) and reinforce how it can then be analyzed as we've done before in our ROOT or python containers**


{% include links.md %}

