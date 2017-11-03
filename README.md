# Datasets and R scripts for best-answers prediction in Technical Q&A sites

## Original data dumps
Original dumps refer to the data extracted "as is" from the following technical Q&A sites:
* Modern platforms
  * [Stack Overflow (SO)](https://www.stackoverflow.com) 
  * [Yahoo! Answer (YA)](https://answers.yahoo.com/dir/index?sid=396545663&link=list) (category: _Programming & Design_)
  * [SAP Community Network (SCN)](https://www.sap.com/community.html) (topics: _Hana_, _Mobility_, _NetWeaver_, _Cloud_, _OLTP_, _Streaming_, _Analytic_, _PowerBuilder_, _Cross_, _3d_, _Frontend_, _ABAP_)
* Legacy platforms
  * [Dwolla](https://discuss.dwolla.com/c/api-support) (discontinued, read-only)
  * [Docusign](https://www.docusign.com) (discontinued, unavailable)

### Download
Data dumps and the description of their file formats are available be downloaded from the [here](https://github.com/collab-uniba/dataset_best-answers_emse/tree/master/dumps).

## Experimental datasets
The datasets containing the features extracted from the data dump of each Q&A site are available for download [here](https://github.com/collab-uniba/dataset_best-answers_emse/tree/master/input).
A description of each feature is also avaialble.

## Python and  R scripts
### Setup
To ensure proper execution, first run the following commands to check for the presence and eventually install all the required packages for R and Python.
```
$ RScripts requirements.R
$ pip install -r requirements.txt
```

### Automated param tuning
To start the automated parameter tuning via `caret`, run the `run-tuning.sh` script as described below. 
```
$ run-tuning.sh models_file data_file
```
* The `models_file` param indicates the file containing (one per line) a list of models (learners) to be tuned. See the file `models/models.txt` for an example.
* The `data_file` param indicates the file containing the data to be used for the tuning stage.
* As output, a TXT file will be created under the `output/tuning/` subfolder for each tuned model, containing the best param configuration and execution times.

_Note_. The tuning step is very time consuming and will take _several_ hours for each model; the more models in the input file, the longer the script will take to finish.

### Scott-Knott ESD model clustering 
To cluster model by AUC performance into non-overlapping groups, run the following scripts:
```
$ python collect-metrics.py --in path/to/metrics/folder.txt --out outfile --ext file_extension --sep field_sep --runs N
```
* `path/to/metrics/folder.txt` - where the tuning script stored the execution log per model for each run 
* `outfile` - the name of file where to store the following main metrics per model per run:
  - AUC
  - F1
  - G-mean
  - Balance
  - Time taken
* `file_extension` - the extension of the output file, chosen in `{txt, csv, xls}`
* `field_sep` - the character used to separate fields in the output file, either `,` or `;`
* `N`- the number of runs used in the tuning step (e.g., 10, 100)

```
$ Rscript skesd-test metrics_outfiles runsN 
```
* `metrics_outfiles` - the file with metrics generated by the Python script at the previous step
* `runsN` - the number of runs, must match the same param from the previous step 

### Feature selection
The following script perform wrapper-based feature selection using the R package `Boruta`; for the sake of completeness, it will also perform Correlation-based Feature Selection (CFS).
```
$ Rscript feature-selection.R dataset_file dataset_name featN
```
* `dataset_file` - the dataset used for feature selection
* `dataset_name` - the name of the dataset, chosen in `{so, docusign, dwolla, scn, yahoo}`; `so` by default
* `featN` - the number of feature to select, 10 by default
* As output, the script will generate the file `output/feature-selection/feature-subset.txt` containing:
  * The output of `Boruta`
  * The output of `CFS`, with both Spearman and Pearson correlation values

### Prediction experiment
Once the models have been tuned, you can execute the best-answer prediction experiment. Run the `run-predictions.sh` script as described below. 
```
$ run-predictions.sh training_file models_file data_file
```
* The `training_file` param indicates the file containing the dataset for training the learners.
* The `models_file` param indicates the file containing (one per line) a list of models (learners) to be used in the prediction experiment. 
* The `data_file` param indicates the file containing the test dataset.
* As output, the following folder and files will be created:
  * `output/cm` - containing a TXT file for each test set and model with the confusion matrix
  * `output/misclassifications` - containing a TXT file for each test set and model with listing the cases where wrong predictions (errors) occurred
  * `output/plots` - containing a ROC plot image file for each test set and model specified as input

_Note_. Before running the prediction experiment, the file `test.R` must be manually edited in order customize the `tuneGrid` var (`dataframe`) containing the best param configuration for each learner model. As of now, the script contains the grids for the 4 models in the file `models/top-cluster.txt`.
