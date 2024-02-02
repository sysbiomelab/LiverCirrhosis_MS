## Reactobiome and reaction richness for Liver Cirrhosis gut microbiome samples

### Prerequisites
Before starting, make sure you have the following:

- MATLAB installed on your system.
- Add the MIGRENE Toolbox directory to your MATLAB path ([here](https://github.com/sysbiomelab/MIGRENE/blob/master/README.md?plain=1#L5)).
- Download the repository or the files i.e., [BacterialAbundace_LiverCirrhosis.xlsx](LiverCirrhosis_MS/BacterialAbundace_LiverCirrhosis.xlsx), [GEMmodels.zip](LiverCirrhosis_MS/BacterialAbundace_LiverCirrhosis.xlsx), [metadata_LiverCirrhosis.txt](LiverCirrhosis_MS/BacterialAbundace_LiverCirrhosis.xlsx), and save them to the same directory on your computer. unzip `GEMmodels.zip`. 

### Load Bacterial Abundance

Load bacterial abundance data from an Excel file

```matlab
cd 'to\your\local\directory'
[abundance,infoFile,~]=xlsread('BacterialAbundace_LiverCirrhosis.xlsx');;
modelList = infoFile(2:end, 1);
sampleName = infoFile(1, 2:end);

%check the samples,
% Remove the MSP name if the abundance of bacteria is zero in all samples
abundance=abundance(sum(abundance,2)~=0,:);
modelList=modelList(sum(abundance,2)~=0,:);
% Remove the samples if there is no bacterial abundance
sampleName=sampleName(sum(abundance,1)~=0);
abundance=abundance(:,sum(abundance,1)~=0);
```
 
### Generate microbiome reaction richness.

Provide the path where the models are saved and the name of the model assigned in the .mat files as follows:
```matlab
PathToModels.path='.\GEMmodels';
PathToModels.name='model';
```
Generate the gut microbiome reaction richness for all individuals

```matlab
richness= RxnRichnessGenerator(modelList,PathToModels,abundance,sampleName);
```

Box plot of reaction richness for normal and disease samples.

```matlab
metadata=readtable('metadata_LiverCirrhosis.txt');
meta1 = table2array(metadata(:,2));
meta1 = char(meta1);
eichnessValue=richness{:,2};
boxplot(eichnessValue,meta1)
title('Reaction richness')
```
![richness](https://github.com/sysbiomelab/LiverCirrhosis_MS/assets/63523016/1318094f-4ed8-4f8e-8872-36fe4acfc79d)


### Generate reactobiome.

```matlab
Reactobiome= ReactobiomeGenerator(modelList,PathToModels,abundance,sampleName);
```

