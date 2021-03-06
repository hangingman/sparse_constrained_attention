# Sparse and Constrained Attention for Neural Machine Translation
----------------
by Chaitanya Malaviya, Pedro Ferreira, André Martins

#### This repository contains scripts for carrying out the experiments described in ref. [1] below. 

### Prerequisites

All procedures are supposed to do in this project root directory:

- fast_align (https://github.com/clab/fast_align)

```
$ sudo apt-get install libgoogle-perftools-dev libsparsehash-dev
$ git clone https://github.com/clab/fast_align.git
$ cd fast_align
$ mkdir build && cd build && cmake .. && make
```

- Python 3.6.10
  - torch==0.3.1
  - torchtext==0.2.3

```
$ pip install -r requirements.txt
```

- OpenNMT-py Unbabel fork (instructions below)
  - I modified OpenNMT-py for python 3.6, you can use it
  - https://github.com/hangingman/OpenNMT-py.git

```
$ git clone -b dev https://github.com/hangingman/OpenNMT-py.git
```

- Prepare WMT2016 dataset
  - https://github.com/rsennrich/wmt16-scripts
  - will be described in next chapter... 


### Preparing the data for a language pair

For the sake of an example, let's assume we're handling the `ro-en` language pair.
The procedure below works for other language pairs, provided the file names
are consistent.

1. Store your data (tokenized and BPE'ed) in a data folder `<DATA PATH>/ro-en`.

- You can download and preprocess dataset with https://github.com/rsennrich/wmt16-scripts
  - Checkout wmt16-scripts, then execute `download_files.sh` and `preprocess.sh`
  - At least, you will find files; `corpus.bpe.ro`, `corpus.bpe.en`, `newsdev2016.bpe.ro`, `newsdev2016.bpe.en`
  - I can't find `newstest2016.*` files but it looks script working 

This must contain the following files:
- `corpus.bpe.ro`
- `corpus.bpe.en`
- `newsdev2016.bpe.ro`
- `newsdev2016.bpe.en`
- `newstest2016.bpe.ro`
- `newstest2016.bpe.en`
- `newstest2016.tc.en`

Note 1: don't forget to include the non-BPE's test target file.

Note 2: these files should *not* include the `sink` symbol.

2. Run the following script:

```
>> prepare_experiment.sh ro en
```

This will add the `sink` symbol on the source files, train the aligner,
force alignments on the dev and test data, and run scripts which
create the gold, guided, actual, and predicted fertility files.
It will also run the `preprocess.py` OpenNMT-py script to create
the train and validation data files.

Note 1: you need to adjust the DATA, ALIGNER, and OPENNMT paths in this script.

Note 2: you need to adjust the PATH_FAST_ALIGN in the script `force_align.py`.

Note 3: you need to adjust the DATA path in the script `fertility/train_test_fertility_predictor.sh`.

### Training models

For training models with different configurations, run the
following example scripts:

```
>> run_experiment.sh <gpuid> ro en softmax 0 &
>> run_experiment.sh <gpuid> ro en sparsemax 0 &
>> run_experiment.sh <gpuid> ro en csoftmax 0 fixed 2 &
>> run_experiment.sh <gpuid> ro en csparsemax 0 fixed 2 &
>> run_experiment.sh <gpuid> ro en csoftmax 0 fixed 3 &
>> run_experiment.sh <gpuid> ro en csparsemax 0 fixed 3 &
>> run_experiment.sh <gpuid> ro en csoftmax 0.2 fixed 3 &
>> run_experiment.sh <gpuid> ro en csparsemax 0.2 fixed 3 &
>> run_experiment.sh <gpuid> ro en csoftmax 0 guided &
>> run_experiment.sh <gpuid> ro en csparsemax 0 guided &
>> run_experiment.sh <gpuid> ro en csoftmax 0.2 guided &
>> run_experiment.sh <gpuid> ro en csparsemax 0.2 guided &
```

This will generate log files in the folder `logs`.

Note: you need to adjust the DATA, ALIGNER, and OPENNMT paths in this script.

### Evaluation

To measure the BLEU and METEOR model performance on the test files,
pick the <model> with best dev performance and use this script:

```
>> ./evaluate.sh <model> <DATA>/ro-en/newstest.bpe.sink.ro <DATA>/ro-en/newstest.tc.en
```

To measure coverage-related metrics (REP-score and DROP-score) on the test files,
please follow the instructions in the README in the `coverage-eval` folder.

In addition, to dump the attention matrices, use the -dump_attn argument with translate.py. You can load the outputted file with pickle as:
```
>> attn_matrices = pickle.load( open(<filename>, 'rb') )
```


## References

[1] Chaitanya Malaviya, Pedro Ferreira, Andre Martins. Sparse and Constrained Attention for Neural Machine Translation. Proceedings of the 56th Annual Meeting of the Association for Computational Linguistics (ACL). Melbourne, Australia, July 2018.


