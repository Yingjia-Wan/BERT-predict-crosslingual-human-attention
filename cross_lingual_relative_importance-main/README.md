The main body of the project codes comes from the Github project: https://github.com/felixhultin/cross_lingual_relative_importance.
However, some modifications are made in order to get the script running:
# Debugging Notes

### 0. supplmented data:  
  set up the geco_ch folder with two subfolders: 'raw' and relfix
  
  downloaded ChineseMaterials.xlsx and L1ReadingData.xlsx in the raw folder
  
  added modelpath and corpus of geco_ch in extract_all.py and analyze_all.py
  
  added two empty txt documents in the result folder:geco_ch_relfix_averages.txt, geco_ch_sentences.txt for writing

### 1. modified configurations (in analysis.py + create_plots.py): 

def process_tokens

line34
original:
doc = Doc(nlp.vocab, words=tokens)
modified:
doc = spacy.tokens.doc.Doc(nlp.vocab, words=tokens)

needed to install spacy models for Chinese tokenization: `python -m spacy download zh_core_web_trf`

### 2. modifed extract_all.py
  removed unnecessary BERT variants
  
  added and edited modelpaths for chinese BERT
  
  included chinese in `def extract_all_human_importance(corpus)`
  
  import data extractor for chinese

### 3. ~modified data_extractor_geco~ created `data_extractor_geco_ch.py`

   ###### ~modified data_extractor_geco by adding `if` for Chinese data extracting~ (**outdated***)
   
  *created a `data_extractor_geco_ch.py` specifically for chinese. Two reasons for  this:
    (1)so that we can test and prcoess the chiense corspus separately;
    (2)to conveniently modified the extracting column name
    
   ###### In `data_extractor_geco`
    
    edited configurations for Chinese
    
    changed the extracted columns representing the same values as in the Chinese data:
      `WORD-TOTAL-READING-TIME` -> `IA_DWELL_TIME`
      `PP_NR` ->  `PP_ID`

### 4. modified analyse_all.py
  added chinese corpus and modelpath, added `if` for Chinese
  
  import `data_extractor_geco`

### 5. cleaned all the files for unneeded languages: Russion, Zuco

  
### 6. `data_extractor_geco_ch.py` : separately run data_extractor_geco_ch before extract_all to write the two files with tokenized data: eco_ch_relfix_averages.txt, geco_ch_sentences.txt (*)

- data_extractor_geco_ch can be called to run in the extract_all.py. HOwever, the calling function is commented in extract_all.py, as we suggest to extract the corpus data first to gain relfix and sentence. This is because of two reasons:

    (1) both data_extractor_geco_ch and extract_saliency/attention take exceptionally long; (E.G., REAIDNG GECCO_CH CORPUS FILE??? ABOUT 10 MINUTES??? EXTRACT_SALIENCY FOR ONE CORPUS: 4H)
    
    (2) due to variations in corpus configuration, error easily occur in data_extractor of a new corpus.

- debugging: 
    import error: need to pip install openpyxl
    
    filenotfound error: modified the reading sentence function by changing the file destination to the abosolute path
    
    filenotfound errors: this happens constantly when we independently run the data_extractor_geco_ch separately. Needed to change all the destination to ones with the absolute path to avoid trouble.
    
    type error in line 58:  subj_data = pd.read_csv(dir+file, delimiter=',') -> subj_data = pd.read_csv(relfix_dir + file, delimiter=',') [THIS SEEMS TO BE A TYPO FROM THE ORIGINAL CODE.]
    
 
    
### 7.  Add `extract_attention_1st_layer.py` in `extract_human_importance folder`:

copied the `extract_attention.py` and modified the copy to create `extract_attention_1st_layer.py`. (The codes are originally adapted from: https://github.com/jessevig/bertviz/)

modified extract_all.py to include extract_attention_1st_layer.py

line110. added:
```
        # ORiginally missing from the source code https://github.com/felixhultin/cross_lingual_relative_importance
        print("Extracting attention_1st_layer for " + modelname)
        extract_all_attention(model, tokenizer, sentences, outfile+ "attention_1st_layer.txt")
```
line 21, added:
`def extract_all_attention_1st_layer(model, tokenizer, sentences, outfile)`

### 8. `analyze_all.py` 
Debugging:

(1) Spacy FUnction TypeError:

Traceback (most recent call last):
  File "analyze_all.py", line 405, in <module>
    human_words_df, aligned_words_df = populate_dataframes(corpora_modelpaths, types)
  File "analyze_all.py", line 126, in populate_dataframes
    pos_tags, frequencies = process_tokens(et_tokens, lang)
  File "/media/isshiki/T7S2TB/alisa/L2-English-eyetracking-data-in-predicting-human-processing/cross_lingual_relative_importance-main/analysis/create_plots.py", line 38, in process_tokens
    processed = nlp(doc)
  File "/home/isshiki/anaconda3/envs/alisa/lib/python3.8/site-packages/spacy/language.py", line 437, in __call__
    doc = self.make_doc(text)
  File "/home/isshiki/anaconda3/envs/alisa/lib/python3.8/site-packages/spacy/language.py", line 467, in make_doc
    return self.tokenizer(text)
TypeError: Argument 'string' has incorrect type (expected str, got spacy.tokens.doc.Doc)
  
 **original code** ("create_plots.py", line 38)
```
doc = Doc(nlp.vocab, words=tokens)
processed = nlp(doc)
```
 **changed to:**
```
doc = Doc(nlp.vocab, words=tokens)
text = ' '.join([token.text for token in doc])
processed = nlp(text)
```
(2) Value error: Unmatached arrays at:

line 145 `sent_df = pd.DataFrame(data, columns=human_words_df.columns)`
line 193 `sent_df = pd.DataFrame(data, columns=aligned_words_df.columns)`

This is because the two objects of pd.DataFrame() have different numbers of columns. 

Solutions: add corresponding keys and values to 'data'. One obstacle is that we don't have 'sentences' variable available. We can create one by modifying the definition of extract_human_importance() function: this function reads from the sentences.txt and store it in the variable `sentences`, therefore we can add the returning value of 'sentences' there, and then define the 'sentences' variable  to add it to 'data' before line 145.
  
(3) (optional improvement) Moved this to when creating dataframes.
  line 153 and 204 now, analyze_all.py
  ```
  human_words_df['length'] = pd.to_numeric(human_words_df['length'])
    human_words_df['frequency'] = pd.to_numeric(human_words_df['frequency'])
  
    aligned_words_df['length'] = pd.to_numeric(aligned_words_df['length'])
    aligned_words_df['frequency'] = pd.to_numeric(aligned_words_df['frequency'])
  ```
(4) LookupError: No wordlist 'best' available for language 'ch'
  line
  
**Problem**: Be careful what abrivation Spacy model library and wedfreq libray use to represent languages when you need to enter it for downloading. e.g., Chinese is 'zh' in wordfreq and pacy, but marked as 'ch' in out script.
  
** Solution**: Added an `if` loop to discuss CHinese and other languages for collecting POS and word frequency.
  
(5) OSError: [Errno 22] Invalid argument: 'results/all_results-2023-01-25-05:58:39.xlsx'
  line 366, analyze_all.py
  
  `timestr = time.strftime("%Y-%m-%d-%H:%M:%S")`
  to
  `timestr = time.strftime("%Y-%m-%d-%H-%M-%S")`
