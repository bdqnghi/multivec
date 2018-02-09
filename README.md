# MultiVec
C++ implementation of *word2vec*, *bivec*, and *paragraph vector*.

## Features

### Monolingual model
* Most of word2vec's features [1, 6]
* Evaluation on the *analogical reasoning task* (multithreaded version of word2vec's *compute-accuracy*)
* Batch and online paragraph vector [2]
* Save & load full model, including configuration and vocabulary
* Python wrapper

### Bilingual model
* Bivec-like training with a parallel corpus [3, 7]
* Save & load full model
* Trains two monolingual models, which can be exported and used by MultiVec
* Python wrapper

## Dependencies
* G++ 4.4+
* CMake 2.6+
* Cython and NumPy for the Python wrapper

## Installation

    git clone https://github.com/eske/multivec.git
    ./compile.sh

The `bin` directory should now contain 4 binaries:
* `multivec` which is used to generate monolingual models;
* `multivec-bi` to generate bilingual models;
* `word2vec` which is a modified version of word2vec that matches our user interface;
* `analogy` to evaluate word embeddings on the analogical reasoning task (multithreaded version of word2vec's compute-accuracy program).

## Usage examples
First create two directories `data` and `models` at the root of the project, where you will put the text corpora and trained models.
The script `scripts/prepare-data.py` can be used to pre-process a corpus (punctuation normalization, tokenization, etc.)

    mkdir data models
    wget http://www.statmt.org/wmt14/training-parallel-nc-v9.tgz -P data
    tar xzf data/training-parallel-nc-v9.tgz -C data
    scripts/prepare-data.py data/training/news-commentary-v9.fr-en data/news fr en --tokenize --normalize-punk --lowercase

To train a monolingual model using text corpus `data/news.en` (use `-v` option to see the training progress):

    bin/multivec --train data/news.en --save models/news.en.bin --threads 16

To train a bilingual model using parallel corpus `data/news.fr` and `data/news.en`:

    bin/multivec-bi --train-src data/news.fr --train-trg data/news.en --save models/news.fr-en.bin --threads 16

To load a bilingual model and export it to source and target monolingual models:

    bin/multivec-bi --load models/news.fr-en.bin --save-src models/news.fr.bin --save-trg models/news.en.bin

To evaluate a trained English model on the analogical reasoning task, first export it to the word2vec format, then use `analogy`:

    bin/multivec --load models/news.en.bin --save-vectors models/vectors.txt
    bin/analogy models/vectors.txt word2vec/questions-words.txt

With this example, you should obtain results similar to these:

    Vocabulary size: 23004
    Embeddings size: 100
    Total accuracy:     11.3% (1157/10195)
    Syntactic accuracy: 12.4% (996/8060)
    Semantic accuracy:  7.54% (161/2135)
    Balanced score:     10.1% (Syntactic: 10.6%, Semantic: 9.71%)

### Python wrapper

    cd cython
    make

Use and train models with Python (`multivec.so` must be in the `PYTHONPATH`, e.g. working directory):

    python3
    >>> from multivec import MonolingualModel, BilingualModel
    >>> model = BilingualModel('../models/news.fr-en.bin')
    >>> model.trg_model
    <multivec.MonolingualModel at 0x7fcfe0d59870>
    >>> model.trg_model.word_vec('france')
    array([ 0.2600708 ,  0.72489363, ...,  1.00654161,  0.38837495])
    >>> new_model = BilingualModel(dimension=300, threads=16)
    >>> new_model.train('../data/news.fr', '../data/news.en')
    >>> help(BilingualModel)  # all the help you need

## Acknowledgements

This toolkit is part of the project KEHATH (https://kehath.imag.fr) funded by the French National Research Agency.

## LREC Paper

When you use this toolkit, please cite:

    @InProceedings{MultiVecLREC2016,
    Title                    = {{MultiVec: a Multilingual and Multilevel Representation Learning Toolkit for NLP}},
    Author                   = {Alexandre Bérard and Christophe Servan and Olivier Pietquin and Laurent Besacier},
    Booktitle                = {The 10th edition of the Language Resources and Evaluation Conference (LREC 2016)},
    Year                     = {2016},
    Month                    = {May}
    }

## References
1. [Distributed Representations of Words and Phrases and their Compositionality](http://arxiv.org/abs/1310.4546), Mikolov et al. (2013)
2. [Distributed Representations of Sentences and Documents](http://arxiv.org/abs/1405.4053), Le and Mikolov (2014)
3. [Bilingual Word Representations with Monolingual Quality in Mind](http://stanford.edu/~lmthang/bivec/), Luong et al. (2015)
4. [Learning Distributed Representations for Multilingual Text Sequences](http://www.aclweb.org/anthology/W15-1512), Pham et al. (2015)
5. [Word2vec project](https://code.google.com/p/word2vec/)
6. [Bivec project](http://stanford.edu/~lmthang/bivec/)
7. [Moses scripts](http://www.statmt.org/moses/)
