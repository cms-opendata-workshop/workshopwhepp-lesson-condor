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
$ ssh userXX@ui3.indiacms.res.in  # replace XX with the number provided to you
~~~
{: .language-bash}

The [example scripts for this HTCondor exercise](https://github.com/ats2008/condorLite/tree/main) were prepared by one of the student facilitators, Aravind Sugunan. Download the scripts from Github:
~~~
$ git clone https://github.com/ats2008/condorLite
$ cd condorLite
~~~
{: .language-bash}

## Condor executable

HTCondor jobs execute a **bash script** on the workder node for each job. This script is called `templates/runScript.tpl.sh`. It is a template with many dummy variables that will be overwritten when a specific copy is made for each job.

The `apptainer` (previously `singularity`) software can be used to access software containers and execute scripts within those containers. Our condor executable has the following basic outline:
- Set environment variables and create a working directory
- Write a bash script containing POET analysis commands
- Execute the analysis script using `apptainer`

### POET analysis commands

The analysis script to be run inside the container is found in the middle of `runScript.tpl.sh`:

~~~
cd /code
source /cvmfs/cms.cern.ch/cmsset_default.sh
scram p CMSSW_7_6_7
cd CMSSW_7_6_7/src/
git clone -b 2015MiniAOD https://github.com/ats2008/PhysObjectExtractorTool.git
scram b -j 4
cmsenv    # Note: perform git access before this command!
cd $_CONDOR_SCRATCH_DIR
cmsRun /code/CMSSW_7_6_7/src/PhysObjectExtractorTool/PhysObjectExtractor/python/poet_cfg.py  @@ISDATA inputFiles=@@FNAMES maxEvents=@@MAXEVENTS outputFile=outfile_@@IDX.root tag=@@TAG
pwd
ls
cp *.root $DESTINATION  
exit 1
~~~
{: .language-bash}

Inside the Open Data docker container, this script will set up the CMS environment and create a CMSSW_7_6_7 software area, similar to what is done when you open the
container using `docker` on your own computer. The script will then clone the 2015 branch of the POET repository from Github and use
`cmsRun` to run POET with several arguments:
- the data or simulation flag
- an input file list
- the maximum number of events to process (-1 for all events)
- the output file name, tagged with a job number
- a ``tag", or label, that you can use to mark this job in any way you wish

Finally, the script copies the POET output ROOT file to a specified destination directory.

> ## I want to edit POET, what do I do?
> Editing the POET configuration, for instance to apply a trigger filter or other event selection, is a 
> great way to reduce the size of the output ROOT files! We recommend developing your POET revisions on
> your own computer, pushing them to your own fork of the POET Github repository, and cloning that version
> in the analysis commands here.
{: .callout}

> ## Clusters without interactive file system mounts
> Some Linux clusters do not provide access to the interactive user file systems on the worker nodes.
> If this is the case on your home cluster, the `cp` command can be replaced with a command appropriate for
> your system.
{: .callout}

### Execution with apptainer

Since the `cmsRun` command is not accessible in the operating system of the worker node, this prepared set of analysis
commands is run using `apptainer`:
~~~
cat container_runScript.sh  # this will display all the commands to be run in apptainer
chmod +x container_runScript.sh
apptainer exec --writable-tmpfs --bind $_CONDOR_SCRATCH_DIR --bind workdir/:/code --bind @@DESTINATION docker://cmsopendata/cmssw_7_6_7-slc6_amd64_gcc493 ./container_runScript.sh
~~~
{: .language-bash}

The arguments include binding (similar to -v or --volume docker argument) several directories to which `apptainer` will have access. These directories include the working directory, output directory and the condor base directory. The other argumnets include dockerhub URL of the container needed for 2015 Open Data analysis, and the analysis script that is to be run inside the container.  

> ## What if my cluster doesn't have Apptainer?
> Consult with your computing system administrators to see if apptainer can be installed on the cluster.
> Follow our [lesson on using Google Cloud resources](https://cms-opendata-workshop.github.io/workshop2023-lesson-cloud/) for an alternative method of running over full datasets.
{: .callout}

### Can I do more than run POET?

Of course! In this example we have chosen to simply produce a POET root file from MiniAOD input files.
You can check out additional code repositories (note: do this before the `cmsenv` command, which affects the git path settings) and execute further analysis commands after creating a POET
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

- The official [HTCondor Manual Quickstart Guide](https://htcondor.readthedocs.io/en/latest/users-manual/quick-start-guide.html) provides a good overview of the condor job submission syntax.
- **FIXME: The specific options configured on the TIFR cluster are described here (ex: which queues are available, other than ``longlunch"?)**


### Arguments to configure

Our condor submission script can accept several arguments:
~~~
parser = argparse.ArgumentParser()
parser.add_argument('-s',"--submit", help="Submit file to condor pool", action='store_true' )
parser.add_argument('-r',"--resubmit", help="Re-Submit file to condor pool", action='store_true' )
parser.add_argument('-t',"--test", help="Test Job", action='store_true' )
parser.add_argument(     "--isData"     , help="is the job running over datafiles ?", action='store_true' )
parser.add_argument('-j',"--njobs", help="Number of jobs to make",default=-1,type=int)
parser.add_argument('-n',"--nFilesPerJob", help="Number of files to process per job",default=1,type=int)
parser.add_argument('-e',"--maxevents", help="Number of events per job",default=-1, type=int)
parser.add_argument('-f',"--flist", help="Files to process",default=None)
parser.add_argument(     "--recid", help="recid of the dataset to process",default=None)
parser.add_argument("--run_template", help="RunScript Template",default='')
parser.add_argument("--tag", help="Tag or vesion of the job",default='condor')

args = parser.parse_args()
~~~
{: .language-python}

In order to submit jobs you will need to provide either a text file with a list of Open Data ROOT files to process or the ``recid" of an Open Data dataset. You can create such a filelist by looking up your dataset on the Open Data Portal webpage or by using the command line tool presented in the dataset scouting lesson. The recid for any dataset can be found in the URL of that dataset on the portal website, and the `cernopendata-client` command line interface will be used to access the list of files for that dataset. All of the other arguments have a default value that you can configure as desired.

> ## Installing cernopendata-client
> On TIFR, `cernopendata-client` is accessible by default. To install it on your own system, see the 
> [installation instructions](https://cernopendata-client.readthedocs.io/en/latest/installation.html) on the 
> client's user manual website. The installation can be checked by running: 
> ~~~
> $ ./local/bin/cernopendata-client version
> ~~~
> {: .language-bash}
> **Note:** you may need to edit `makeCondorJobs.py` to point to `./local/bin/cernopendata-client/`.
>
> **Note:** the instructions assume that python3 is the default python program on the system. If your system
> has python3 available but uses python2 as the default, use `pip3 install` in place of the generic `pip install`.
{: .callout}

### Prepare tailored executable files

After parsing the user's command-line arguments, and optionally generating a file list from `cernopendata-client`, the script will prepare individual executable files for each
job. The specified number of files per job will be taken in sequence from the file list, an output file directory will be prepared, and all of the dummy values in the template executable that begin with `@@` will be overwritten:
~~~
print(f"Making Jobs in {runScriptTemplate} for files from {filelistName}")
jobid=0
while fileList and jobid < args.njobs: 
    jobid+=1
    flsToProcess=[]
    for i in range(args.nFilesPerJob):
        if not fileList:
            break
        flsToProcess.append(fileList.pop())

    fileNames=','.join(flsToProcess)
    dirName  =f'{head}/Job_{jobid}/'
    if not os.path.exists(dirName):
        os.system('mkdir -p '+dirName)
    if not os.path.exists(destination):
        os.system('mkdir -p '+destination)

    runScriptName=dirName+f'/{htag}_{jobid}_run.sh'
    if os.path.exists(runScriptName+'.sucess'):
       os.system('rm '+runScriptName+'.sucess')
    runScript=open(runScriptName,'w')
    tmp=runScriptTxt.replace("@@DIRNAME",dirName)
    tmp=tmp.replace("@@TAG",str(args.tag))
    tmp=tmp.replace("@@ISDATA",str(args.isData))
    tmp=tmp.replace("@@PWD",pwd)
    tmp=tmp.replace("@@IDX",str(jobid))
    tmp=tmp.replace("@@FNAMES",fileNames)
    tmp=tmp.replace("@@MAXEVENTS",str(args.maxevents))
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
print("Condor Jobs can now be submitted by executing : ")
for fle in allCondorSubFiles:
    print('condor_submit '+fle)
    if args.submit or args.resubmit:
        os.system('condor_submit '+fle)
print("")
~~~
{: .language-python}


{% include links.md %}

