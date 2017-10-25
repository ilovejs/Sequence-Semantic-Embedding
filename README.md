# Sequence-Semantic-Embedding

SSE(Sequence Semantic Embedding) is an encoder framework toolkit for natural language processing related tasks and it's implemented in TensorFlow by leveraging TF's convenient deep learning blocks like DNN/CNN/LSTM etc. 

SSE model translates a sequence of symbols into a vector of numeric numbers, so that different sequences with similar semantic meanings will have closer numeric vector distances. This numeric number vector is called the SSE for the original sequence of symbols. SSE can be applied to some large scale NLP related machine learning tasks. For example:
* **Text classification task**: e.g., mapping a listing title or search query to one or multiples of the 20,000+ leaf categories in eBay website.
* **Search engine relevance ranking task**: e.g., mapping a search query to some most relevant documents in the inventory.
* **Question answering task**: e.g.,  mapping a question to its most suitable answers.
* **Cross lingual information retrieval task**: e.g., mapping a Chinese/English/Mixed-Language search query to its most relevant English documents in inventory without translating Chinese to English.

Depending on each specific task, similar semantic meanings can have different definitions. For example, in the category classification task, similar semantic meanings means that for each correct pair of (listing-title, category), the SSE of title is close to the SSE of corresponding category.  While in the information retrieval task, similar semantic meaning means for each relevant pair of (query, document), the SSE of query is close to the SSE of relevant document. While in the question answering task, the SSE of question is close to the SSE of correct answers.

This repo contains some sample raw data, tools and recipes to allow user establish complete End2End solutions from scratch for four different typical NLP tasks: text classification, relevance ranking, cross-language information retrieval and question answering. This includes deep learning model training/testing pipeline, index generating pipeline, command line demo app, and the run-time RESTful webservices with trained models for these NLP tasks. By replacing supplied raw data with your own data, users can easily establish a complete solution for their own NLP tasks with this deep learning based SSE tech.

Here is the quick-start instruction to build a text classification webservice from scratch including download the repo, setup environment, train model, run demo app and setup webserver.


```bash
git clone https://github.com/eBay/Sequence-Semantic-Embedding.git
cd Sequence-Semantic-Embedding
./env_setup.sh
make train-classificastion
make demo-classification
export FLASK_APP=webserver.py
export MODEL_TYPE=classification
python -m flask run --port 5000 --host=0.0.0.0
```

Once webserver has started, you can open a browse and send a GET request like: http://your-computer-ip-address:5000/api/classify?keywords=sunglasses


See the [Content](#content) below for more details on how SSE training works, and how to use it to build the complete solution for your own NLP task with your own dataset.

### [Contents](#content)

* [Setup Instructions](#setup)
* [SSE Model Training](#modeltraining)
    * [Basic Idea](#idea)
    * [Training Data](#data)
    * [Train Model](#train)
    * [Visualize Training progress in Tensor Board](#tensorboard)
* [Command Line Demo](#demo)
* [Setup WebService](#service)
* [Call WebService to get results](#callservice) 
* [Build Your Own NLP task services with your own data](#customizebuild)
    * [text classification](#classification)
    * [search engine relevance ranking](#ranking)
    * [cross-lingual information retrieval](#crosslingual)
    * [question answering](#qna)
* [References](#reference)

---

### Setup Instructions

The code of SSE toolkit support both python2 and python3. Just issue below command to download the repo and install dependencies such as tensorflow.

``` bash
git clone https://github.com/eBay/Sequence-Semantic-Embedding.git
cd Sequence-Semantic-Embedding
./env_setup.sh
```

### SSE Model Training

## Basic Idea

SSE encoder framework supports three different types of network configuration modes: source-encoder-only, dual-encoder and shared-encoder. 

* In source-encoder-only mode, SSE will only train a single encoder model(RNN/LSTM/CNN) for source sequence. For target sequence, SSE will just learn its sequence embedding directly without applying any encoder models. This mode is suitable for closed target space tasks such as classification task, since in such tasks the target sequence space is limited and closed thus it does not require to generate new embeddings for any future unknown target sequences outside of training stage. A sample network config diagram is shown as below:
    ![computation graph](images/Source-Encoder-OnlyModeForSSE.png)

* In dual-encoder mode, SSE will train two different encoder models(RNN/LSTM/CNN) for both source sequence and target sequence. This mode is suitable for open target space tasks such as relevance ranking, cross-lingual information retrieval, or question answering, since the target sequence space in those tasks is open and dynamically changing, a specific target sequence encoder model is needed to generate embeddings for new unobserved target sequence outside of training stage. A sample network config diagram is shown as below:
    ![computation graph](images/Dual-EncoderModeforSSE.png)

* In shared-encoder mode, SSE will train one single encoder model(RNN/LSTM/CNN) shared for both source sequence and target sequence. This mode is suitable for open target space tasks such as question answering system or relevance ranking system, since the target sequence space in those tasks is open and dynamically changing, a specific target sequence encoder model is needed to generate embeddings for new unobserved target sequence outside of training stage. In shared-encoder mode, the source sequence encoder model is the same as target sequence encoder mode. Thus this mode is better for tasks where the vocabulary between source sequence and target sequence are similar. A sample network config diagram is shown as below:
    ![computation graph](images/shared-encoderModeForSSE.png)

## Training Data

This repo contains some sample raw datasets for four types of NLP task in below four subfolders:

* **rawdata-classification**: The unziped DataSet.tar.gz contains three text files named as TrainPairs, EvalPairs and targetIDs. The targetIDs file contains target's category_class_name and category_class_ID, seperated by tab. The TrainPair and EvalPair have the same data format for training corpus and evaluation corpus. The file format is source_sequence(listing title), target_class_sequence(category name), target_class_id(category id), seperated by tab. 
    
* **rawdata-qna**: The unziped DataSet.tar.gz contains three text files named as TrainPairs, EvalPairs and targetIDs. The targetIDs file contains all possible answer document's whole content and the answer_document_id, seperated by tab. The TrainPair and EvalPair have the same data format for training corpus and evaluation corpus. The file format is source_sequence(question), target_sequence(answer_document_content), target_sequence_id(answer_document_id), seperated by tab. 

* **rawdata-searchranking**: The unziped DataSet.tar.gz contains three text files named as TrainPairs, EvalPairs and targetIDs. The targetIDs file contains the search inventory's listing_title and listing_id, seperated by tab. The TrainPair and EvalPair have the same data format for training corpus and evaluation corpus. The file format is source_sequence(search query), target_sequence(listing title), target_sequence_id(listing id), seperated by tab. 

* **rawdata-crosslingual**: The unziped DataSet.tar.gz contains three text files named as TrainPairs, EvalPairs and targetIDs. The targetIDs file contains the English language based search inventory's listing_title and listing_id, seperated by tab. The TrainPair and EvalPair have the same data format for training corpus and evaluation corpus. The file format is source_sequence(Chinese/English/Mixed-Languages search query), target_sequence(English listing title), target_sequence_id(English listing id), seperated by tab.  


## Train Model

To start training your models, issue command as below. For classification task, use option of *train-classification* ; for question answer task, use option of *train-qna*, for relevance ranking task use option of *train-ranking*, for cross-lingual task use option of *train-crosslingual*

``` bash
make train-classification

```

## Visualize Training progress in Tensor Board

Use below command to start TF's tensorboard and view the Training progress in your webbrowser.

``` bash
$> tensorboard --logdir=models-classification

```

## Command Line Demo

Use below command to start the demo app and then follow the prompt messages. 

``` bash
make demo-classification

```

## Setup WebService

``` bash
export SSE_MODEL_DIR=models-classification
export FLASK_APP=webserver.py
python -m flask run --port 5000 --host=0.0.0.0

```

## Call WebService to get results

Once the webserver starts, you can use below curl command to test its prediction results for your NLP task.

``` bash
curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET http://<your-ip-address>:5000/api/classify?keywords=men's running shoes

```
or you can just open a web browser and put a GET request like below to see your NLP task service result:

```
http://<your-ip-address>:5000/api/classify?keywords=men's running shoes
```


### Build Your Own NLP task services with your own data

##  text classification

The mission of text classification task is to classify a given source text sequence into the most relevant/correct target class(es). If there is only one possible correct class label for any given source text, i.e., the class labels are exclusive to each other, this is called single-label text classification task. If there are multiple correct class labels can be associated with any given source text, i.e., the class labels are independent to each other, this is called multiple-label text classification task.

The current supplied sample classification raw dataset is from single-label text classification task. A newer version sample data for multiple-label text classification raw dataset will be released in near future.

If you want to train the model and build the webservice with your own dataset from scratch,  you can simply replace zip file in rawdata-classification folder with your own data and keep the same data format specified in [Training Data](#data) section. And then issue below commands to train out your own model, build the demo app and setup your webservice:


```bash
make train-classificastion
make demo-classification
export FLASK_APP=webserver.py
export MODEL_TYPE=classification
python -m flask run --port 5000 --host=0.0.0.0
```

Once the webserver starts, you can just open a web browser and put a GET request like below to see your text classification service result:

```
http://<your-ip-address>:5000/api/classify?keywords=men's running shoes
```

The webserver will return a json object with a list of top 10 (default) most relevant target class with ID, names and matching scores.

## question answering

The mission of question answering task is to provide the most relevant answer for a given question. The example we provided here is for simple question answering scenarios.  We have a set of FAQ answer documents, when a user asking a question, we provide the most relevant FAQ answer document back to the user.

If you want to build your own question answering webservice with your own FAQ dataset from scratch,  you can simply replace the zip file in rawdata-qna folder with your own data and keep the same data format specified in [Training Data](#data) section. And then issue below commands to train out your own model, build the demo app and setup your webservice:


```bash
make train-qna
make demo-qna
export FLASK_APP=webserver.py
export MODEL_TYPE=qna
python -m flask run --port 5000 --host=0.0.0.0
```

Once the webserver starts, you can just open a web browser and put a GET request like below to see your question answering web service result:

```
http://<your-ip-address>:5000/api/qna?question=how does secure pay work&?nbest=5
```

The webserver will return a json object with a list of top 5 (default) most relevant answer document with document_ID, document_content and matching scores.


## search engine relevance ranking

To be added very soon.

## cross-lingual information retrieval

To be added very soon.

## References
More detailed information about the theory and practice for deep learning(DNN/CNN/LSTM/RNN etc.) in NLP area can be found in papers and tutorials as below:

 * [ Deep Learning for Natural Language Processing: Theory and Practice (Tutorial) ](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/CIKM14_tutorial_HeGaoDeng.pdf)
 * [ Learning Semantic Representations Using Convolutional Neural Networks for Web Search ](https://www.microsoft.com/en-us/research/publication/learning-semantic-representations-using-convolutional-neural-networks-for-web-search/)
 * [ Deep Sentence Embedding Using LSTM Networks: Analysis and Application to Information Retrieval ](http://arxiv.org/abs/1502.06922)
 * [ Sequence to Sequence Learning with Neural Networks ](http://arxiv.org/abs/1409.3215)
 * [ Neural Machine Translation by Jointly Learning to Align and Translate ](http://arxiv.org/abs/1409.0473)
 * [ On Using Very Large Target Vocabulary for Neural Machine Translation ](http://arxiv.org/abs/1412.2007)
 * [ Tensorflow's Machine Translation Implementation based on Seq2Seq model ](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/models/rnn/translate)
 * [ SubwordText tokenizer from Tensor2Tensor project](https://github.com/tensorflow/tensor2tensor/blob/master/tensor2tensor/data_generators/text_encoder_build_subword.py)