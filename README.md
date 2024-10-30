# HiForestSetupPbPbRun2024
Instructions on how to setup and run the rapid validation forests for 2024 PbPb and pp reference runs.

## Setup environment:
Setup on LXPlus8 for CMSSW_14_1_X.
```bash
cmsrel CMSSW_14_1_4_patch2
cd CMSSW_14_1_4_patch2/src
cmsenv
git cms-merge-topic CmsHI:forest_CMSSW_14_1_X
git remote add cmshi git@github.com:CmsHI/cmssw.git
scram build -j8
voms-proxy-init --voms cms
```

For production, it is a good idea to create a production branch and commit all the changes every time before running production, so that the exact configuration used for the production can be traced back if needed. 
```bash
git checkout -b rapidValidationForest2024
```

## Link your private repository 

You will also need to fork the [https://github.com/CmsHI/cmssw/tree/forest_CMSSW_14_1_X](CmsHI/cmssw) git repository, which can be done on the GitHub website using the "Fork" dropdown. Keep the name the same.

From the "Branches" tab, add a new branch based on "forest_CMSSW_14_1_X" and name it "rapidValidationForest2024".

Finally, create the link to your private repo.
```bash
cd CMSSW_14_1_4_patch2/src/HeavyIonsAnalysis/Configuration/test/

# Create a remote alias for your repo
git remote add my-cmssw git@github.com:<your_github_username>/cmssw.git

# Check that it has been added:
git config -l
# You should also see remote repos for 'official-cmssw' and 'cmshi'

# Do a fetch to get the list of branches
git fetch my-cmssw

# Push to establish the link for the 'rapidValidationForest2024' branch
git push my-cmssw rapidValidationForest2024:rapidValidationForest2024
```

## Test the configuration

To test the configuration, you will first need to obtain a sample input file from the dataset you are running over. Easy way to get a file list for a dataset of your choice is the following:

```bash
ls /eos/cms/store/group/phys_heavyions/wangj/RECO2023/miniaod_PhysicsHIPhysicsRawPrime0_374322/* > fileListHIPhysicsRawPrime0_run374322.txt
sed -i -e "s#/eos/cms#root://eoscms.cern.ch/#" fileListHIPhysicsRawPrime0_run374322.txt
```

Then update this input file to the the configuration file
```bash
vim forest_miniAOD_run3_DATA.py
```
Now you can run the configuration interactively to ensure that it works for your file:
```bash
cmsRun forest_miniAOD_run3_DATA.py
```

## Submit the jobs with crab

To submit the jobs via CRAB, copy the template file without emap from this area to the CMSSW area where you are running the jobs. The file with emap is included as an example if similar trick with ZDC needs to be done again as was done in 2023. You will first need to modify the template file according to your needs
```
vim crabForestTemplate.py
```
Modify the following lines:
```python
# At the top:
inputList
jobTag

# Check that memory and run time settings are good:
config.JobType.maxMemoryMB
config.JobType.maxJobRuntimeMin

# Write the output somewhere where you have write rights:
config.Site.storageSite
```
Notice that you will need a file list of all the files in the dataset in .txt format for the inputList variable. The jobTag can be whichever name that describes the job you are sending.

Once you have customized the CRAB template file, for documentation purposes, commit the changes note the commit you are using to submit the jobs
```bash
git add -u
git commit -m "Descriptive comment"
git push my-cmssw rapidValidationForest2024:rapidValidationForest2024
```
Then remember the commit hash
```bash
git rev-parse HEAD > gitCommitLog.txt
```

After the bookkeeping is done, submit the jobs.
```bash
crab submit -c crabForestTemplate.py
```

When the jobs are finished, you should document in the Twiki page https://twiki.cern.ch/twiki/bin/view/CMS/HiForest2024 that the forest is done, and for each forest add the git link for the configuration that was used to create the said forest. Example link for a commit looks like this: https://github.com/jusaviin/cmssw/tree/250185e12b917d36fd8d3a51208e5f8311f3ad92.

CRAB likes to create a long structure of unnecessary folders for the output files. It is a good idea to trunkate this structure, since CRAB also has a lenght limit for input files, and a long file structure might cause errors in subsequent job submission. The following command will do the trick:
```bash
eos file rename /eos/cms/store/group/phys_heavyions/jviinika/run3RapidValidation/PbPb2023_run374322_HIExpressRawPrime_withDFinder_2023-09-27/CRAB_UserFiles/crab_PbPb2023_run374322_HIExpressRawPrime_withDFinder_2023-09-27/230928_014852/0000 /eos/cms/store/group/phys_heavyions/jviinika/run3RapidValidation/PbPb2023_run374322_HIExpressRawPrime_withDFinder_2023-09-27/0000
```

## Helpful scripts

There are a couple of helpful bash scripts included in this directory that can make life easier in certain occasions. If you want to create a forest using prompt reconstruction files for certain run, you can easily create a file list with ```findFilesForRun.sh``` by defining primary dataset and run number. Running it without arguments print usage help.

Another script is for merging all .root files in a specific folder, that can be either local or at CERN EOS. This is called ```addHistograms.sh```. Again, running the script without arguments gives you usage instructions for this script.

If you are running several CRAB jobs at the same time, you can use the ```checkCrabJobs.sh``` script to easily check the statuses for all of them. It takes a text file with all job names as an input, and outputs only the status information about all the jobs. For more details, run the script without arguments.
