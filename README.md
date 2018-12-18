                                                 ONLINE DEMO COMING SOON....

# SOURCES
We’ve taken pre-trained embeddings from BERT model - https://github.com/google-research/bert           
Automatic Q/A generation from TheGadFlyProject - https://github.com/TheGadflyProject/TheGadflyProject                 
- REQUIREMENTS
You’ll need TensorFlow, Spacy, Python 3.6, nltk           
# INTRODUCTION
Due to high volume of data on the internet, it is becoming increasingly difficult to search for relevant information. Search engines can be  useful to find such information but sometimes it is difficult to answer question asked in natural language. Question-Answering systems for large datasets can be easily trained to give great results but training a deep neural network on a small dataset leads to poor results. One such example is a Question-Answering system for Stony Brook University. To develop QA system for such a scenario, we propose methods to learn weights from other large datasets and then fine-tune it using Stony Brook University website data. We built QA system for SBU using BERT base pre-trained weights. We used several other techniques to retrieve correct context paragraphs and evaluate our system. 

We took pre-trained weights of BERT base system and then fine-tuned it with SQuAD QA training data. To decide paragraph of answer we used multiway classification and infersent.

Most of our work is in 3 files - 
- doc_classifier.ipnb               
- fine_tuning_squad.ipynb                     
- fine_tunign_sbu_squad.ipynb                    
### doc_classifier.ipnb (Jupyter notebook)                                             
  For finding context of a given question we wrote the doc_retrieval module, it has jupyter notebook which generates json that            can be passed to the trained model for prediction
### fine_tuning_squad.ipynb (colab notebook)                                                                  
In the BERT module, we load the pre-trained embedding and run run_squad.py which was given with the BERT repository.
Also, you can run the colab notebook  - https://colab.research.google.com/drive/1vaWITP0EmlmDn-bxAckzrxM8T_gbCAdj
### DownloadWebsite.py
This file is responsible to download wikipedia data of stony brook university and it will convert data such that it can be during pre-training.

# SETUP
We use Google collaboratory to explore the BERT model and out experiments went well so we decided to fine tune our network on colab, so the it does not matter where you keep files locally, you need to upload them to the notebooks directory structure or upload it to your drive and then mount your drive. The folder BERT/bert_reqs contains all the requirements you'll need, so make sure you're using it correctly in colab. There are 2 options - 
- Upload file directly from local system
- Mount Google cloud on Colab and Upload files from Google cloud (RECOMMENDED)           
We recommend using cloud, because we experienced that it was faster and it's frustruating to browse for requirements everytime the runtime enviroment resets or notebook resets.
# FAQs 
### How can I train using run_squad.py?                      
You can fine tune your own network with pre-trained embeddings on SQUAD using the following command - 
This command is also present in fine_tuning_squad.ipynb                   
```
python run_squad.py \
  --vocab_file=$BERT_BASE_DIR/vocab.txt \
  --bert_config_file=$BERT_BASE_DIR/bert_config.json \
  --init_checkpoint=$BERT_BASE_DIR/bert_model.ckpt \
  --do_train=True \
  --train_file=train-v1.2.json \
  --do_predict=True \
  --predict_file='handmade_qa_sbu.json' \
  --train_batch_size=8 \
  --learning_rate=3e-5 \
  --num_train_epochs=2.0 \
  --max_seq_length=384 \
  --doc_stride=128 \
  --output_dir=$OUTPUT_DIR 
  ```
### How can I pre-train using stony brook data?     
We can use sbu_small_pretrain.tfrecord generated by downloadData.py to pre-train bert_base. Here is a command to do that. 
  ```
python run_pretraining.py \
  --input_file=sbu_small_pretrain.tfrecord \
  --output_dir=./temp/ \
  --do_train=True \
  --do_eval=True \
  --bert_config_file=bert_config.json \
  --init_checkpoint=bert_model.ckpt \
  --train_batch_size=32 \
  --max_seq_length=128 \
  --max_predictions_per_seq=20 \
  --num_train_steps=20 \
  --num_warmup_steps=10 \
  --learning_rate=2e-4
  ```
### How can I predict my own questions and contexts?   
You just need to generate json from doc_classifier.ipynb by passing the list of question and context to the create_json() methond and then run the fine tuned network for prediction.            
NOTE: Please note that output folder must not be empty and should contain checkpoint and data files  
#### Generating SQUAD style json file - 
```
#You can also generate json for you own context and paragraphs
custom_para = ['''Domestic Student Health Insurance Plan (SHIP) .Benefits and Highlights of the SHIP.SHIP 
has been developed especially for Stony Brook students (and their dependents) to provide access to comprehensive
care that complements the quality health services on campus.The details of the plan are reviewed and recommended
each year by committee members to ensure that the coverage is well-suited to the needs of the Stony Brook students 
and respectful of their budgets. SHIP is administered by United Healthcare. The Plans meet all of the student health
insurance standards developed by the American College Health Association.SHIP is tailor-made for the college
population.Provides continuous coverage at a reasonable cost for most on or off-campus medical care over 
Fall/Winter and Spring/Summer Semesters.Covers pre-existing medical conditions & preventative care.
Annual deductible $200 for an individual.Annual out of pocket limit of $3,000 which includes deductibles, 
copays and coinsurance.Covers inpatient and outpatient mental health care.No deductible applied to prescription 
drug coverage.Please note: Office visits for Primary Care and Specialists have a $35 copayment 
with 0% coinsurance with a referral and 30% coinsurance without a referral.''']
custom_ques = ["What is the annual deductible amount for SHIP?"]
json_file = create_json(custom_para,custom_ques)
```

We used this to find context paragraph containing answer. Basic indea is question embedding should be closer to the paragraph containing the answer. Primary code change is in Infersent/encoder/Infersent.ipynb.


#### Generating predictions for this file
```
python run_squad.py \
  --vocab_file=$BERT_BASE_DIR/vocab.txt \
  --bert_config_file=$BERT_BASE_DIR/bert_config.json \
  --init_checkpoint=$BERT_BASE_DIR/bert_model.ckpt \
  --do_train=False \
  --train_file=train-v1.2.json \
  --do_predict=True \
  --predict_file='qna_sbu_test.json' \
  --train_batch_size=8 \
  --learning_rate=3e-5 \
  --num_train_epochs=2.0 \
  --max_seq_length=384 \
  --doc_stride=128 \
  --output_dir=$OUTPUT_DIR 
  ```
#### See your answer -
```
!cat output_small/predictions.json
```
### How can I test on your hand annotated test json file?  
You can test our system on fine-tuned network with the hand annotated dataset with the following command
```
python run_squad.py \
  --vocab_file=$BERT_BASE_DIR/vocab.txt \
  --bert_config_file=$BERT_BASE_DIR/bert_config.json \
  --init_checkpoint=$BERT_BASE_DIR/bert_model.ckpt \
  --do_train=False \
  --train_file=train-v1.2.json \
  --do_predict=True \
  --predict_file='handmade_qa_sbu.json' \
  --train_batch_size=8 \
  --learning_rate=3e-5 \
  --num_train_epochs=2.0 \
  --max_seq_length=384 \
  --doc_stride=128 \
  --output_dir=$OUTPUT_DIR 
  ```
### TODO
- [x] Fine tune BERT on SQUAD                                                          
- [x] Create document classifier to get context                                               
- [ ] ADD DEMO                                                

