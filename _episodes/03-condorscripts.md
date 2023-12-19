---
title: "HTCondor submission"
teaching: 20
exercises: 0
questions:
- "How can I use the CMSSW docker container in a condor job?"
- "How can I divide a dataset up into several jobs?"
- "How do I track the progress of condor jobs?"
objectives:
- "Understand the example scripts for running POET in a container on condor"
- "Learn the basics of submitting and monitoring HTcondor jobs"
keypoints:
- "The condor job control file can specify a docker container."
- "Each job's executable file can specify code to access from Github to perform an analysis task."
- "References are included here to condor submission and monitoring guides."
---

## Download example scripts

As you tested earlier in the workshop, please log in to the TIFR cluster:
~~~
$ ssh userXX@ui3.indiacms.res.in
~~~
{: .language-bash}

The example scripts for this HTCondor exercise were prepared by one of the student facilitators, Aravind Sugunan. Download the scripts from Github:
~~~
$ git clone https://github.com/ats2008/condorLite
$ cd condorLite
~~~
{: .language-bash}

## Condor executable

HTCondor jobs execute a **bash script** on the workder node for each job. This script is called `templates/runScript.tpl.sh`. It is a template with many dummy variables that will be overwritten when a specific copy is made for each job.

### Apptainer setup

The `apptainer` (previously `singularity`) software can be used to access software containers and execute scripts within those containers

**FIXME: I'm not sure whether or not we will need to have them do any apptainer commands before submitting?**

**FIXME: otherwise, we'll point out how `runScript.tpl.sh` sets up access to a container? If there's nothing needed beyond the apptainer exec line I described below we can remove this section and introduce apptainer below.**

### POET

The exectuable will clone the 2015 branch of the POET repository and use `cmsRun` to run POET with several arguments:
- an input file list
- the maximum number of events to process (-1 for all events)
- the output file name, tagged with a job number


~~~
cd $_CONDOR_SCRATCH_DIR
FIXME: git clone -b 2015MiniAOD https://github.com/cms-opendata-analyses/PhysObjectExtractor.git
cmsRun /code/CMSSW_7_6_7/src/PhysObjectExtractorTool/PhysObjectExtractor/python/poet_cfg.py  inputFiles=@@FNAMES maxEvents=@@MAXEVENTS outputFile=outfile_@@IDX.root
~~~
{: .language-cpp}

The POET output ROOT file will be copied to a specified destination directory:
~~~
ls  # this will show the produced ROOT files
cp *.root $DESTINATION
~~~
{: .language-cpp}

> ## Clusters without interactive file system mounts
> Some Linux clusters do not provide access to the interactive user file systems on the worker nodes.
> If this is the case on your home cluster, the `cp` command can be replaced with a command appropriate for
> your system.
{: .callout}

### Execution with apptainer

Since the `cmsRun` command is not accessible in the operating system of the worker node, this prepared set of
commands is run using `apptainer`:
~~~
cat container_runScript.sh  # this will display all the commands to be run in apptainer
chmod +x container_runScript.sh
apptainer exec --writable-tmpfs --bind $_CONDOR_SCRATCH_DIR --bind $BASEWDIR:/code --bind @@DESTINATION docker://cmsopendata/cmssw_7_6_7-slc6_amd64_gcc493 ./container_runScript.sh
~~~
{: .language-cpp}

### Can I do more than run POET?

Of course! In this example we have chosen to simply produce a POET root file from MiniAOD input files.
You can check out additional code repositories and execute further analysis commands after creating a POET
root file.

> ## Do I need apptainer after POET?
> Maybe not. Python-based analysis scripts can likely be run directly on a condor worker node if the
> native python distribution can provide the packages you want to use.
> However, the `apptainer` execution command shown here for the CMSSW container can be adapted to
> use either the ROOT or Python containers that you have seen in this workshop.
{: .callout}

## Submitting the condor jobs

Now we will explore the condor submission script in `scripts/makeCondorJobs.py`. At the top of this script
is a template for a **job control file**:
~~~
condorScriptString="\
executable = $(filename)\n\
output = $Fp(filename)run.$(Cluster).stdout\n\
error = $Fp(filename)run.$(Cluster).stderr\n\
log = $Fp(filename)run.$(Cluster).log\n\
+JobFlavour = \"longlunch\"\n\
"
~~~
{: .language-python}

Condor job control files have lines that configure various job behaviors, such as which executable file to send to the worker node and what names to use for output and error files. Different condor clusters will allow different specific lines for requesting CPU or memory, for directing jobs to different queues within the cluster, for passing command line arguments to the executable, etc.

The official [HTCondor Manual Quickstart Guide](https://htcondor.readthedocs.io/en/latest/users-manual/quick-start-guide.html) provides a good overview of the condor job submission syntax.

### Arguments to configure

Our condor submission script can accept several arguments:
~~~
parser = argparse.ArgumentParser()
parser.add_argument('-s',"--submit", help="Submit file to condor pool", action='store_true' )
parser.add_argument('-r',"--resubmit", help="Re-Submit file to condor pool", action='store_true' )
parser.add_argument('-t',"--test", help="Test Job", action='store_true' )
parser.add_argument('-p',"--printOnly", help="Only Print the commands", action='store_true' )
parser.add_argument('-j',"--njobs", help="Number of jobs to make",default='-6000')
parser.add_argument('-n',"--nFilesPerJob", help="Number of files to process per job",default=1,type=int)
parser.add_argument('-e',"--nevts", help="Number of events per job",default='-1')
parser.add_argument('-f',"--flist", help="Files to process",default=None)
parser.add_argument("--run_template", help="RunScript Template",default='')
parser.add_argument('-v',"--version", help="Vesion of the specific work",default='TEST')

args = parser.parse_args()
~~~
{: .language-python}

In order to submit jobs you will need to provide a text file with a list of Open Data ROOT files to process (**FIXME: switch to recid?**). You can create such a filelist by looking up your dataset on the Open Data Portal webpage or by using the command line tool presented in the dataset scouting lesson. All of the other arguments have a default value that you can configure as desired.

### Prepare tailored executable files

After parsing the user's command-line arguments, the script will prepare individual executable files for each
job. The specified number of files per job will be taken in sequence from the file list, an output file directory will be prepared, and all of the dummy values in the template executable that begin with `@@` will be overwritten:
~~~
    print(f"Making Jobs in {runScriptTemplate} for files from {args.flist}")
    jobid=0
    while fileList and jobid < njobs: 
        jobid+=1
        flsToProcess=[]
        for i in range(nfilesPerJob):
            if not fileList:
                break
            flsToProcess.append(fileList.pop())

        fileNames=','.join(flsToProcess)
        dirName  =f'{head}/Job_{jobid}/'
        if not os.path.exists(dirName):
            os.system('mkdir -p '+dirName)
        destination=f'{RESULT_BASE}/{JOB_TYPE}/{job_hash}/'
        if not os.path.exists(destination):
            os.system('mkdir -p '+destination)
 
        runScriptName=dirName+f'/{htag}_{jobid}_run.sh'
        if os.path.exists(runScriptName+'.sucess'):
           os.system('rm '+runScriptName+'.sucess')
        runScript=open(runScriptName,'w')
        tmp=runScriptTxt.replace("@@DIRNAME",dirName)
        tmp=tmp.replace("@@PWD",pwd)
        tmp=tmp.replace("@@IDX",str(jobid))
        tmp=tmp.replace("@@FNAMES",fileNames)
        tmp=tmp.replace("@@MAXEVENTS",str(maxevents))
        tmp=tmp.replace("@@RUNSCRIPT",runScriptName)
        tmp=tmp.replace("@@DESTINATION",destination)
        runScript.write(tmp)
        runScript.close()
        os.system('chmod +x '+runScriptName)
~~~
{: .language-python}

### Submit the jobs

Finally, the individual condor job control scripts are prepared to point to specific executable files, and
the `condor_submit` command is called to submit each control script to the condor job queue.
~~~
print("All condor submit files to be submitted ")
for fle in allCondorSubFiles:
    print('condor_submit '+fle)
    if submit2Condor or resubmit2Condor:
        os.system('condor_submit '+fle)
print("")
~~~
{: .language-python}

## Monitoring condor jobs

HTCondor supports many commands that can provide information on the status of job queues and a user's submitted jobs. Details are availabel in the HTCondor manual for [managing a job](https://htcondor.readthedocs.io/en/latest/users-manual/managing-a-job.html). Three extremely useful commands are shared here.

To see the status of your jobs:
~~~
$ condor_q
~~~
{: .language-bash}

**FIXME: let's show some output for a set of running POET jobs specifically on this cluster**

This command shows the cluster identification number for each job, its idle (I) / held (H) / running (R) status,
the current run time, and the command that was run.

To remove a job that you would like to kill:
~~~
$ condor_rm CLUSTERID
~~~
{: .language-bash}

**FIXME: is that enough for this cluster, or does it need --name QUEUE? let's also add output**

To watch the progress of a job that is ongoing:
~~~
$ condor_tail -f CLUSTERID
~~~

**FIXME: is that enough for this cluster, or does it need --name QUEUE? let's also add output**

{% include links.md %}

