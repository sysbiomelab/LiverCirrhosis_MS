Before starting, make sure you have the following:

- MATLAB installed on your system.
- Add the MIGRENE Toolbox directory to your MATLAB path ([here](https://github.com/sysbiomelab/MIGRENE/blob/master/README.md?plain=1#L5)).
- Download liver_cirrhosis.zip file and unzip it in your local directory.

### Setting up Paths

 Specify paths to necessary files and directories.
 
 ```matlab
% get path to where the MIGRENE Toolbox is located
MIGDIR = fileparts(which('MIGRENE_pipeline'));
% provide the path to bacterial species (MSP) gene info and bacterial
% abundance obtained from metagenomics analysis
CATDIR=[MIGDIR filesep 'data'];
ABUNDANCE=[CATDIR filesep 'BacterialAbundance.xlsx'];
PATHWAY=[CATDIR filesep 'pathways.xlsx'];
% provide the path to microbiomeGEM (generated in
% IntegrationCatalogToModel.m) and bibliome data.
MATDIR=[MIGDIR filesep 'mat'];
MODEL=[MATDIR filesep 'microbiomeGEM.mat'];
BIBLIOME=[MATDIR filesep 'bibliome.mat'];
% define a directory to save microbiomeGEM
SAVEDIR=[MIGDIR filesep 'saveDir'];
% number of cores specified for parallelization. it can be a positive
% integer or a range specified as a 2-element vector of integers
numWorkers=4;
```

### Loading Bacterial Abundance

Load bacterial abundance data from Excel file

```matlab
[abundance, infoFile, ~] = xlsread(ABUNDANCE);
modelList = infoFile(2:end, 1);
sampleName = infoFile(1, 2:end);

%check the samples,
% Remove the MSP name if the abundance of bacteria in all samples is zero
abundance=abundance(sum(abundance,2)~=0,:);
modelList=modelList(sum(abundance,2)~=0,:);
% Remove the samples if there is no bacterial abundance
sampleName=sampleName(sum(abundance,1)~=0);
abundance=abundance(:,sum(abundance,1)~=0);
```


### Initializing COBRA Toolbox
Initiate the COBRA Toolbox for constraint-based modeling.

```matlab
initCobraToolbox()
 ```
 
## Tutorial Overview

The tutorial consists of the following main steps:
1. Generating microbiome reaction richness.
2. Generating reaction abundance.
3. Generating reactobiome.
4. Generating reaction set enrichment.
5. generating community models.

## Tutorial Details
### Generating microbiome reaction richness.

Please provide the path where the models are available and the name of the model assigned in the .mat files.

```matlab
PathToModels.path=SAVEDIR;
PathToModels.name='contextSpecificModel';
% generate gut microbiome reaction composition (reaction richness) of all individuals 
richness= RxnRichnessGenerator(modelList,PathToModels,abundance,sampleName);

```

### Generating reaction abundance.

generate reaction abundance for all individuals; the function generates both reaction abundance and relative reaction abundance

```matlab
[reactionRelativeAbun, rxnAbunPerSample]= ReactionAbundanceGenerator(modelList,PathToModels,abundance,sampleName);
```

### Generating reactobiome.

```matlab
Reactobiome= ReactobiomeGenerator(modelList,PathToModels,abundance,sampleName);
```

### Conducting enrichment analysis.

You need to prepare the following files:

1) a file includes pathway terms and the IDs that could be KO, EC,kegg reaction ID, etc. Here we use the KEGG pathway terms with Kegg reaction ID

```matlab
[~,terms,~]=xlsread(PATHWAY);
terms=terms(2:end,:);
```

2) a file for ID mapping between reaction names in the models and the IDs in the pathway file. here, we use the info in the reference model. 

```matlab
load(MODEL)
IDmap=[microbiomeGEM.rxns microbiomeGEM.rxnRN];
Index = find(not(cellfun('isempty',IDmap(:,2))));
IDmap=IDmap(Index,:);
```

If you assign the p-value, then coverage of non-significance terms is set as zero. if you dont define the p-value, it returns all. 

```matlab
p_value=0.05;
[coverageRSE,pRSE]= pRSEGenerator(modelList,PathToModels,abundance,sampleName,IDmap,terms,p_value);
```

coverageRSE shows the coverage of each pathway in the samples and pRSE shows the p-value of the pathways in the samples

### Generating community models.

Define the number of top abundant bacteria for community modeling. Here we generate communities for top 10 bacteria in each sample.

```matlab
top=10;
thre=[];
for i=1:size(abundance,2)
t1=sort(abundance(:,i),1,'descend');
thre(i,1)=t1(top,1);
abundance(find(abundance(:,i) < thre(i,1)),i)=0;
end
boxplot(thre)
median(thre)
```

Generation of community models

```matlab
%specify the metabolite ID and exchange reaction for biomass (optional)
biomass.EXrxn='Ex_Biomass';
biomass.mets='cpd11416ee[lu]';
% Make a directory to save generated community models
if ~exist([SAVEDIR filesep 'community'],'dir')
mkdir([SAVEDIR filesep 'community']);
end
PathToSave=[SAVEDIR filesep 'community'];
[report]= MakeCommunity(modelList,PathToModels,abundance,sampleName,PathToSave,biomass);
```

In report, "one" shows that a community model has been generated for the individual in PathToSave directory
