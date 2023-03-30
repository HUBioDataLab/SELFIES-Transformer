# SELFIES-Transformer: Learning the Representation of Chemical Space for Drugs Discovery
Chemical and protein text representations can be conceived of as unstructured languages that humans have codified to describe domain-specific information. The use of natural language processing (NLP) to reveal that hidden knowledge in textual representations of these biological entities is at an all-time high. Discovering and developing new drugs is a critical topic considering the fast-growing and aging population and health risks caused by it, such as complex diseases (e.g., types of cancer). Conventional experimental procedures used for developing drugs are expensive, time-consuming and labor-intensive, which in turn decreases the efficiency of this process. In our paper, we proposed an NLP model that uses a large-scale pre-training methodology on 2 million molecules in their SELFIES representation to learn flexible and high-quality molecular representations for drug discovery problems, followed by a fine-tuning process to support varied downstream molecular analysis tasks such as molecular property prediction. As a result, our model outperformed [ChemBERTa](https://arxiv.org/abs/2010.09885) on all molecular analysis tasks and was only marginally behind [MolBERT](https://arxiv.org/abs/2011.13230) with good model interpretation ability. We hope that our strategy will reduce costs in the bioinformatics field, allowing researchers to continue their research without the need for additional funding.

## The Transformer Architecture
Our pre-trained model is implemented as RobertaMaskedLM. We then achieve sequence outputs as the model's output to use for molecule representations. These representations will be used for fine-tuning. In order to use the sequence output for visualisation, we will be taking average of the sequence output.

Our fine-tuning model’s architecture was based on RobertaForSequenceClassification’s architecture. Our model’s architecture for the fine-tuning process includes a pre-trained RoBERTa model as a base model and RobertaClassificationHead class for the next layers as a classifier. RobertaClassificationHead class consists of a dropout layer, a dense layer, tanh activation function, a dropout layer, and a final linear layer for a classification task or a regression task in this order respectively. We forward the sequence output of the pre-trained RoBERTa base model to the classifier to use during the fine-tuning process for supervised tasks. We can achieve sequence outputs as the fine-tuned models's output for molecule representations. In order to use the sequence output for visualisation, again we will be taking average of the sequence output.


## Getting Started

We highly recommend you to use conda platform for installing dependencies properly. After installation of appropriate conda version for your operating system, create and activate conda environment with dependencies as below:

```
conda create -n selfiesTransformer_env
conda activate selfiesTransformer_env
conda env update --file data/requirements.yml
```

## Generating Molecule Embeddings Using Pre-trained Models
Pre-trained SELFormer models are available for download [here](https://drive.google.com/file/d/1zuVAKXCMc-HZHQo9y3Hu5zmQy51FGduI/view?usp=sharing). Embeddings of all molecules from CHEMBL30 that are generated by our best performing model are available [here](https://drive.google.com/drive/folders/1Ii44Z6HonzJv5B5VYFujVaSThf802e2M?usp=sharing). 

<br/>

You can also generate embeddings for your own dataset using the pre-trained models. To do so, you will need SELFIES notations of your molecules. You can use the command below to generate SELFIES notations for your SMILES dataset.

<br/>

If you want to reproduce our code for generating embeddings of CHEMBL30 dataset, you can unzip __molecule_dataset_smiles.zip__ and/or __molecule_dataset_selfies.zip__ files in the __data__ directory and use them as input SMILES and SELFIES datasets, respectively.

```
python3 generate_selfies.py --smiles_dataset=data/molecule_dataset_smiles.txt --selfies_dataset=data/molecule_dataset_selfies.csv
```

* __--smiles_dataset__: Path of the input SMILES dataset.
* __--selfies_dataset__: Path of the output SELFIES dataset.

<br/>

To generate embeddings for the SELFIES molecule dataset using a pre-trained model, please run the following command:

```
python3 produce_embeddings.py --selfies_dataset=data/molecule_dataset_selfies.csv --model_file=data/pretrained_models/modelO --embed_file=data/embeddings.csv
```

* __--selfies_dataset__: Path of the input SELFIES dataset.
* __--model_file__: Path of the pretrained model to be used.
* __--embed_file__: Path of the output embeddings file.

### Generating Embeddings using pre-trained models for MoleculeNet data
The embeddings generated by our best performing pre-trained model for MoleculeNet data can be directly downloaded [here](https://drive.google.com/drive/folders/1Xu3Q1T-KwXb67MF3Uw63pFm2IzoxeNNY?usp=share_link).

<br/>

You can also re-generate these embeddings using the command below.

```
python3 get_moleculenet_embeddings.py --dataset_path=data/finetuning_datasets --model_file=data/pretrained_models/modelO 
```
* __--dataset_path__: Path of the directory containing the MoleculeNet datasets.
* __--model_file__: Path of the pretrained model to be used.

## Training and Evaluating Models

### Pre-Training
To pre-train a model, please run the command below. If you have a SELFIES dataset, you can use it directly by giving the path of the dataset to __--selfies_dataset__. If you have a SMILES dataset, you can give the path of the dataset to __--smiles_dataset__ and the SELFIES dataset will be created at the path given to __--selfies_dataset__.

```
python3 train_pretraining_model.py --smiles_dataset=data/molecule_dataset_smiles.txt --selfies_dataset=data/molecule_dataset_selfies.csv --prepared_data_path=data/selfies_data.txt --bpe_path=data/BPETokenizer --roberta_fast_tokenizer_path=data/RobertaFastTokenizer --hyperparameters_path=data/pretraining_hyperparameters.yml --subset_size=100000
```

* __--smiles_dataset__: Path of the SMILES dataset. It is required if __--selfies_dataset__ does not exist. Optional.
* __--selfies_dataset__: Path of the SELFIES dataset. If a SELFIES dataset does not exist, it will be created at the given path using the __--smiles_dataset__. If it exists, SELFIES dataset will be used directly. Required.
* __--prepared_data_path__: Path of the intermediate file that will be created during pre-training. It will be used for tokenization. If it does not exist, it will be created at the given path. Required.
* __--bpe_path__: Path of the BPE tokenizer. If it does not exist, it will be created at the given path. Required.
* __--roberta_fast_tokenizer_path__: Path of the RobertaTokenizerFast tokenizer. If it does not exist, it will be created at the given path. Required.
* __--hyperparameters_path__: Path of the yaml file that contains the hyperparameter sets to be tested. Note that these sets will be tested one by one and not in parallel. Example file is available at /data/pretraining_hyperparameters.yml. Required.
* __--subset_size__: The size of the subset of the dataset that will be used for pre-training. By default, the whole dataset will be used. Optional.

### Fine-tuning on Molecular Property Prediction

You can use commands below to fine-tune a pre-trained model for various molecular property prediction tasks. These commands are utilized to handle datasets containing SMILES representations of molecules. SMILES representations should be stored in a column with a header named "smiles". You can see the example datasets in the __data/finetuning_datasets__ directory. 

<br/>

**Binary Classification Tasks**

To fine-tune a pre-trained model on a binary classification dataset, please run the command below. 

```
python3 train_classification_model.py --model=data/saved_models/modelO --tokenizer=data/RobertaFastTokenizer --dataset=data/finetuning_datasets/classification/bbbp/bbbp.csv --save_to=data/finetuned_models/modelO_bbbp_classification --target_column_id=1 --use_scaffold=1 --train_batch_size=16 --validation_batch_size=8 --num_epochs=25 --lr=5e-5 --wd=0
```

* __--model__: Directory of the pre-trained model. Required.
* __--tokenizer__: Directory of the RobertaFastTokenizer. Required.
* __--dataset__: Path of the fine-tuning dataset. Required.
* __--save_to__: Directory where the fine-tuned model will be saved. Required.
* __--target_column_id__: Default: 1. The column id of the target column in the fine-tuning dataset. Optional.
* __--use_scaffold__: Default: 0. Determines whether to use scaffold splitting (1) or random splitting (0). Optional.
* __--train_batch_size__: Default: 8. Optional.
* __--validation_batch_size__ : Default: 8. Optional.
* __--num_epochs__: Default: 50. Number of epochs. Optional.
* __--lr__: Default: 1e-5: Learning rate. Optional.
* __--wd__: Default: 0.1: Weight decay. Optional.

<br/>

**Multi-Label Classification Tasks**

To fine-tune a pre-trained model on a multi-label classification dataset, please run the command below. The RobertaFastTokenizer files should be stored in the same directory as the pre-trained model.

```
python3 train_classification_multilabel_model.py --model=data/saved_models/modelO --dataset=data/finetuning_datasets/classification/tox21/tox21.csv --save_to=data/finetuned_models/modelO_tox21_classification --use_scaffold=1 --batch_size=16 --num_epochs=25 --lr=5e-5 --wd=0
```

* __--model__: Directory of the pre-trained model. Required.
* __--dataset__: Path of the fine-tuning dataset. Required.
* __--save_to__: Directory where the fine-tuned model will be saved. Required.
* __--use_scaffold__: Default: 0. Determines whether to use scaffold splitting (1) or random splitting (0). Optional.
* __--batch_size__: Default: 8. Train batch size. Optional.
* __--num_epochs__: Default: 50. Number of epochs. Optional.
* __--lr__: Default: 1e-5: Learning rate. Optional.
* __--wd__: Default: 0.1: Weight decay. Optional.

<br/>

**Regression Tasks**

To fine-tune a pre-trained model on a regression dataset, please run the command below. 

```
python3 train_regression_model.py --model=data/saved_models/modelO --tokenizer=data/RobertaFastTokenizer --dataset=data/finetuning_datasets/regression/esol/esol.csv --save_to=data/finetuned_models/modelO_esol_regression --target_column_id=-1 --scaler=2 --use_scaffold=1 --train_batch_size=16 --validation_batch_size=8 --num_epochs=25 --lr=5e-5 --wd=0
```
 
* __--model__: Directory of the pre-trained model. Required.
* __--tokenizer__: Directory of the RobertaFastTokenizer. Required.
* __--dataset__: Path of the fine-tuning dataset. Required.
* __--save_to__: Directory where the fine-tuned model will be saved. Required.
* __--target_column_id__: Default: 1. The column id of the target column in the fine-tuning dataset. Optional.
* __--scaler__: Default: 0. Method to be used for scaling the target values. 0 for no scaling, 1 for min-max scaling, 2 for standard scaling. Optional.
* __--use_scaffold__: Default: 0. Determines whether to use scaffold splitting (1) or random splitting (0). Optional.
* __--train_batch_size__: Default: 8. Optional.
* __--validation_batch_size__ : Default: 8. Optional.
* __--num_epochs__: Default: 50. Number of epochs. Optional.
* __--lr__: Default: 1e-5: Learning rate. Optional.
* __--wd__: Default: 0.1: Weight decay. Optional.
