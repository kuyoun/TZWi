# TZWi
Yet another CMS NanoAOD tools focusing on specific analyses.
  * Management area: https://app.asana.com/0/1115311953973868/list

We'd like to cover:

  * ttbar dilepton analysis
    * inclusive cross section, differential cross section
    * ttbar+bbbar cross section ratio to ttbar+jets
  * ttbar rare decays
    * t->qZ, ttbar and single top

**See also**
- https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookNanoAOD
- NanoAOD dataformat descriptions
  - 2016 samples: https://cms-nanoaod-integration.web.cern.ch/integration/master-102X/mc80X_doc.html
  - 2017 samples: https://cms-nanoaod-integration.web.cern.ch/integration/master-102X/mc94Xv2_doc.html
  - 2018 samples: https://cms-nanoaod-integration.web.cern.ch/integration/master-102X/mc102X_doc.html

## Installation
```bash
cmsrel CMSSW_10_2_11
cd CMSSW_10_2_11/src
cmsenv
git-cms-init
git cms-merge-topic cms-nanoAOD:master-102X
git checkout -b nanoAOD cms-nanoAOD/master-102X
git clone https://github.com/cms-nanoAOD/nanoAOD-tools.git PhysicsTools/NanoAODTools
git clone https://github.com/cms-kr/TZWi
scram b -j
```

## (Optional) Customized NanoAOD production
```bash
cd TZWi/NanoAODProduction/test
./generateConfig.sh ## This will produce 3 sets of 3 cfg files...
crab submit...
```

## List up NanoAOD samples
Update sample list, produce file lists
```bash
tzwi-updatedataset $CMSSW_BASE/src/TZWi/NanoAODProduction/data/datasets/NanoAOD/2016/*.yaml
tzwi-updatedataset $CMSSW_BASE/src/TZWi/NanoAODProduction/data/datasets/NanoAOD/2017/*.yaml
```

## Run postprocessors

Assume we are working at KISTI Tier2/3 and cms-kr/hep-tools package is installed.
```bash
NFILE=5
cd $CMSSW_BASE/src/TZWi/TopAnalysis/test/ttbarDoubleLepton
for MODE in ElEl MuEl MuMu; do
    for FILELIST in NanoAOD/2017/*/*/*.txt; do
        NJOBS=`cat $FILELIST | xargs -n$NFILE | wc -l`
        JOBNAME=$MODE.`basename $FILELIST | sed -e 's;.txt;;g'`
        create-batch bash 01_prod_ntuple.sh $MODE $FILELIST $NFILE --jobName $JOBNAME -T --nJobs $NJOBS
    done
done
```

Wait for the jobs to be finished, check output files, resubmit failed jobs.

Tip to list up failed job and resubmit them:
```bash
for i in *NANOAOD*/; do
    echo $i
    cd $i
    rm -f failed.txt
    for j in result*.tgz; do
      tar tzf $j | grep -q failed.txt && tar -Oxzf $j failed.txt >> failed.txt 2> /dev/null
    done
    cd ..
done

for i in *NANOAOD*/; do
    cd $i
    if [ -f failed.txt ]; then
        rm -f result*.tgz job*.err job*.log failed.txt
        ./submit.sh
    fi
    cd ..
done
```

You can process failed ones manually:
```bash
cat failed.txt | sed 's;nano_postproc.py;;g' | xargs -P$(nproc) -L1 nano_postproc.py
```

Tip to extract all ntuples:
```bash
find *NANOAOD*/ -name 'result_*.tgz' | awk '{print "xzf "$1" ./ntuple"}' | xargs -L1 -P$(nproc) tar
```

## Make histograms
This step will draw all histograms including systematics variations using maximum 20 CPUs in parallel.
```bash
./02_make_histograms.py
```
