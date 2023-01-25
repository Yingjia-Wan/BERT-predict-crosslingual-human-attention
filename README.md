# L2-English-eyetracking-data-in-predicting-human-processing
This project contains the code for the 4k essay by Mphil student Yingjia Wan, Department of Theoretical and Applied Linguistics, University of Cambridge.

### Acknowledgements
A large part of the code base of https://github.com/beinborn/relative_importance has been re-purposed for this project.
The codes for multilingual data_extractor and analysis are based on https://github.com/felixhultin/cross_lingual_relative_importance. Necessary modifications of corpus data and scripts are made by me and recorded in steps at [Debugging History](Debugging History).

### 2 Requirements

Python should be <= 3.8. We use the following packages (see requirements.txt):  
numpy (1.19.5), tensorflow (2.4.1), transformers (4.2.2), scikit-learn (0.22.2), spacy (2.3.5), wordfreq (2.3.2), scipy (1.4.1)
Note that later versions of transformers might lead to errors.

To install, create and activate a virtual environment and run:  
`pip3 install -r requirements.txt`

*If adding a corpus for a different language, you also need to download the language-specific spaCy model either to your virtual environment, or incorporate it in the script, from https://spacy.io/models.
For example, for Chinese, run this before the experiments: `python -m spacy download zh_core_web_sm`.

### 1 Rerunning the experiments
- The complete commands to run the whole experiments (without plots):
```
cd L2-English-eyetracking-data-in-predicting-human-processing/cross_lingual_relative_importance-main
pip3 install -r requirements.txt
python extract_all.py
python analyze_all.py
```
- Extract_all.py takes exceptionally longer time to complete, as it calls to run scripts from both [extract_human_fixations] amd [extract model importance] folder. For example, it takes 3-4 hours to read and tokenize GECO_EN corpus to generate results for human relative fixation importacne. For each model per corpus (e.g., BERT-UNCASED for GECO_EN), it requires 6-7 hours to extract the 1st-layer attention, last-layer attention, and saliency scores. Therefore, results for these human importance and model importance from serveral language corpora are stored in the results file, so you can skip running `python extract_all.py` to save time.

- If running `analyze_all.py` for the first time, the script will take a bit longer to create two `.csv`-files: `aligned_words.csv` and `human_words.csv`. These contain all word-level information needed to run the experiments, e.g. token, relative importance, word length and word frequency. These are saved in the `results/words` folder and can be used for additional analysis.

Once `analyze_all.py` has finished, a final Excel file will be created: `all_results-<timestamp>.xlsx`. It contains all of the results organized into four tabs.

- **Model Importance**: Correlation results (Spearman R) between human and model relative word importance.
- **Permutation Baselines** Correlation results between model relative word importance and random numbers. Used as a sanity check, but not presented in paper.
- **Corpus statistical baselines** Correlation results (Spearman R) between human and model relative word importance and the two corpus statistical baselines: Word frequency and word length.
- **Regression analysis**: Results of the linear regression analysis (out-of-date: look at section 5).  

### 2 Optional preprocessing: extract_all.py
`extract_all.py` is optional if you don't need to change or add any corpus/model, because the intermediate files have already been generated and saved in this repository.

- intermediate files:
`analyze_all.py` (see section 1) require several intermediate result files to run. These can be found in the `results` folder and consist of files with the following formats `<corpus>_<hf_modelpath>_<importance type>.txt`, which align model relative importance to words in a corpus, `<corpus>_relfix_averages.txt`, which align words with human relative importance, and `<corpus>_sentences` which are simply the sentences in the corpus.

These files are created by running `extract_all.py`, which use the files created by the data extractor scripts in the folder `extract_human_fixations`. See its specific [README](extract_human_fixations/README.md) for information on how to add a new corpus or re-run the scripts.


### 3. Adding Corpus and Model:
Specific suggestions and tips of changing the corpus and models in this project code can be found in Debugging History.md.



### 4. Additional fine-grained Analysis: 

- Generating plots
Run `python -m analysis.create_plots all_results-<timestamp>.xlsx` on the Excel file (see section 1) to create the respective plots. The plots are saved in the `plots` folder.

- Regression analysis using Linear mixed models (LMM)

Since LMMs are not readily available in Python, the results of the regression analysis in the paper was done in Stata. To run the script, run `stata mixed-effects/lmm.do`. If you want to create the plots, run `convert_tables_to_results.py`, which will create a `conversion.xlxs` Excel file. Move the `with_reffect` tab of the `conversion.xlsx` Excel file to the `Regression analysis` tab of the original `all_results-<timestamp>.xlsx` Excel file. Then you can run `python -m analysis.create_plots all_results-<timestamp>.xlsx`.

### 5. Folder structure

- **extract_human_fixations**: code to extract the relative fixation duration from the five eye-tracking corpora and average it over all subjects. Corpus data is also saved here. See [README](extract_human_fixations/README.md)

- **extract_model_importance**: code to extract saliency-based, attention-based importance from transformer-based language models.

- **analysis**: code to compare and analyze patterns of importance in the human fixation durations and the model data. Also contains code to replicate the plots in the paper.

- **plots**: contains all plots.

- **results**: contains intermediate results.
