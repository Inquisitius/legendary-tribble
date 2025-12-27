# legendary-tribble
TNT-KID repo with modifications to make the data pipeline for concept extraction easier.
The following code is based off of TNT-KID by Martinc et al. The original README for the TNT-KID repository as well as links to the github are w

Many of the subdirectory are not relevant for our usecase. The ones being utilized are /data and /data_text

Text-to-JSON Preprocessor (txtToJSON.py)
This script converts a folder of .txt files into a single JSON array. It is designed to clean the text and optionally break long files into smaller "Parts" so they fit into the AI model's 512-token limit.

Sample Command:
python txtToJSON.py --input_folder "raw_data" --output_file "data.json" --chunk --chunk_size 512

Argument Descriptions
--input_folder: The path to the folder containing your text files.

--output_file: The name you want for the resulting JSON file (e.g., test.json).

--folder_name (Optional): Overrides the ID prefix. If the folder is "CS449", IDs will be "CS449-1". Using --folder_name Lecture changes them to "Lecture-1".

--chunk: A toggle flag. If included, long files will be split into multiple JSON objects (Part 1, Part 2, etc.).

--chunk_size: The maximum number of words per part. Defaults to 512.

This should generate you a JSON array file that you can run directly with the pretrained model.

Virtual Environment:
run source bin/activate inside of the research folder (Should have all the required packages to run)

How to compile and use the pretrained model:
python predict.py --data_path data/slideTest.json --bpe_model_path bpe/bpe_science.model --dict_path dictionaries/5_lm+bpe+rnn_science_adaptive_lm_bpe_nopos_rnn_nocrf.ptb --trained_classification_model trained_classification_models/model_5_lm+bpe+rnn_science_folder_kp20k.pt --adaptive --rnn --bpe
(Add a --cuda flag at the end if you are using a GPU)
(Alternatively you can switch the --trained_classification_model flag to use the news model included in that directory)


ORIGINAL TNT-KID README:



# Code for experiments conducted in the paper 'TNT-KID: Transformer-based Neural Tagger for Keyword Identification' submitted to Natural Language Engineering journal #

Please cite the following paper [[bib](https://gitlab.com/matej.martinc/tnt_kid/-/blob/master/bibtex.js)] if you use this code:

Martinc, M., Škrlj, B., & Pollak, S. (2021). TNT-KID: Transformer-based neural tagger for keyword identification. Natural Language Engineering, 1-40. doi:10.1017/S1351324921000127


## Installation, documentation ##

Published results were produced in Python 3 programming environment on Linux Mint 18 Cinnamon operating system. Instructions for installation assume the usage of PyPI package manager.<br/>
To get the source code and the train and test data, clone the project from the repository with 'git clone https://gitlab.com/matej.martinc/tnt_kid'<br/>
To only get the source code, clone the repository from github with 'git clone https://github.com/EMBEDDIA/tnt_kid'<br/>

Install dependencies if needed: pip install -r requirements.txt

### To extract keywords with the model trained on scientific texts (KP20k dataset): ###

```
python predict.py --data_path data/example.json --bpe_model_path bpe/bpe_science.model --dict_path dictionaries/5_lm+bpe+rnn_science_adaptive_lm_bpe_nopos_rnn_nocrf.ptb --trained_classification_model trained_classification_models/model_5_lm+bpe+rnn_science_folder_kp20k.pt --adaptive --rnn --bpe --cuda
```

### To extract keywords with the model trained on news (KPTimes dataset): ###

```
python predict.py --data_path data/example_news.json --bpe_model_path bpe/bpe_news.model --dict_path dictionaries/5_lm+bpe+rnn_news_adaptive_lm_bpe_nopos_rnn_nocrf.ptb --trained_classification_model trained_classification_models/model_5_lm+bpe+rnn_news_folder_kptimes.pt --adaptive --rnn --bpe --cuda
```

### To reproduce the results published in the paper run the code in the command line using following commands: ###

Generate news and science datasets for language model training:<br/>
```
python data/build_dataset.py --datasets data --data_path data
```

Train language model on the computer science domain articles:<br/>
```
python train_and_eval.py --config_id 5_lm+bpe+rnn_science --lm_corpus_file data_science.json --bpe_model_path bpe/bpe_science.model --adaptive --rnn --bpe --cuda
```

Train language model on the news articles:<br/>
```
python train_and_eval.py --config_id 5_lm+bpe+rnn_news --lm_corpus_file data_news.json --bpe_model_path bpe/bpe_news.model --adaptive --rnn --bpe --cuda
```

Train and test the keyword tagger on the datasets from the computer science domain:<br/>
```
python train_and_eval.py --config_id 5_lm+bpe+rnn_science --bpe_model_path bpe/bpe_science.model --datasets 'kp20k;inspec;krapivin;nus;semeval' --classification --transfer_learning --rnn --bpe --cuda
```

Train and test the keyword tagger on the datasets from the news domain:<br/>
```
python train_and_eval.py --config_id 5_lm+bpe+rnn_news --bpe_model_path bpe/bpe_news.model --datasets 'kptimes;jptimes;duc' --classification --transfer_learning --rnn --bpe --cuda
```

### Instructions for training the model on a new dataset: ###

The dataset needs to be in the json line format, where each document in the dataset is a json file containing the keys "title", "abstract", and "keywords" (containing keywords separated by semi-column). Example:

{"title": "Title of the text", "abstract": "abstract of the scientific paper or text of the news article ", "keywords": "kw1;kw2;kw2"}

For bpe tokenizer training and language model training, the name of the dataset is arbitrary, but it should be put in the "data" folder. For example, if you named it as "new_dataset.json" then you can run the following commands:

To train bpe tokenizer:

```
python bpe/bpe.py --input new_dataset.json --output new_dataset
```

To train language model:

```
python train_and_eval.py --config_id lm+bpe+rnn_new_dataset --lm_corpus_file new_dataset.json --bpe_model_path bpe/new_dataset.model --adaptive --rnn --bpe --cuda
```

For training of the keyword tagger, the dataset needs to be split into the train and test set and here the naming matters. An example that works would be to create a folder "new_dataset" in the "data" folder. 
Name the train dataset as "new_dataset_valid.json" and the test dataset as "new_dataset_test.json" and put them in the folder "new_dataset". Note that there should be a match between the folder name and train and test datasets names (without the suffixes). 
The suffix "_valid.json" tells the script that this is a train set and the suffix "_test.json" tells the script that this is the test set.

Train and test the keyword tagger on the new dataset:<br/>
```
python train_and_eval.py --config_id lm+bpe+rnn_new_dataset --bpe_model_path bpe/new_dataset.model --datasets 'new_dataset' --classification --transfer_learning --rnn --bpe --cuda
```

## Contributors to the code ##

Matej Martinc<br/>

* [Knowledge Technologies Department](http://kt.ijs.si), Jožef Stefan Institute, Ljubljana
