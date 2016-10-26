
# Link to the CombineHarvester twiki:
[CombineHarvester twiki](http://cms-analysis.github.io/CombineHarvester/index.html)


# Checkout the CombineHarvester
    export SCRAM_ARCH=slc6_amd64_gcc481 (bash) or  setnev SCRAM_ARCH slc6_amd64_gcc481 (tcsh)
    scram project CMSSW CMSSW_7_1_5
    cd CMSSW_7_1_5/src
    cmsenv
    git clone https://github.com/cms-analysis/HiggsAnalysis-CombinedLimit.git HiggsAnalysis/CombinedLimit
    //Check the recommended tag on link above, a tag &gt;= v5.0.2 is sufficient
    git clone https://github.com/cms-analysis/CombineHarvester.git CombineHarvester
    git clone https://github.com/roger-wolf/HiggsAnalysis-HiggsToTauTau-auxiliaries.git auxiliaries [this is not necessary for the time being, but better to have a directory called auxiliaries so it could be a place for the rootfiles ]
    scram b


# Creating the working directory:
    cd CombineHarvester/CombineTools/bin
    wget https://github.com/cms-analysis/CombineHarvester/blob/SM2016-dev/HTTSM2016/HTT.cpp
    // Edit HTT.cpp if needed and add HTT.cpp to the Build File
    scram b
    HTT 


# Building the workspaces:
    cd output/sm_cards
    combineTool.py -M T2W -i {cmb,em,et,mt,tt}/* -o workspace.root --parallel 4


# Calculating limits:
    combineTool.py -M Asymptotic -t -1 -d */*/workspace.root --there -n .limit --parallel 4


# Collect the output:
    combineTool.py -M CollectLimits */*/*.limit.* --use-dirs -o limits.json


# Plotting the limit:
    plotLimits.py limits_{cmb,em,et,mt,tt}.json:exp0 --auto-style

# Making the pulls:
    combine -M MaxLikelihoodFit cmb/125/workspace.root  --robustFit=1 --preFitValue=1. --X-rtd FITTER_NEW_CROSSING_ALGO --minimizerAlgoForMinos=Minuit2 --minimizerToleranceForMinos=0.1 --X-rtd FITTER_NEVER_GIVE_UP --X-rtd FITTER_BOUND --minimizerAlgo=Minuit2 --minimizerStrategy=0 --minimizerTolerance=0.1 --cminFallbackAlgo \"Minuit2,0:1.\"  --rMin 0.5 --rMax 1.5
    
    python ../../../../../HiggsAnalysis/CombinedLimit/test/diffNuisances.py mlfit.root -A -a --stol 0.99 --stol 0.99 --vtol 99. --vtol2 99. -f text mlfit.root &gt; mlfit.txt


# Computing the expected significance:
    combine -M ProfileLikelihood --significance cmb/125/workspace.root -t -1 --expectSignal=1


# Computing the Impact
    combineTool.py -M Impacts -d cmb/125/workspace.root -m 125 --doInitialFit --robustFit 1 -t -1
    combineTool.py -M Impacts -d cmb/125/workspace.root -m 125 --robustFit 1 --doFits -t -1
    combineTool.py -M Impacts -d cmb/125/workspace.root -m 125 -o impacts.json
    plotImpacts.py -i impacts.json -o impacts
