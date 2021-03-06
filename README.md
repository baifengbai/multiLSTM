# multiLSTM for Joint NLU

this implements a recurrent model for joint intent detection and slot-filling for the NLU task.

## requirements

```
gensim==3.4.0
h5py==2.8.0
Keras==2.2.0
keras-contrib==2.0.8
keras-utilities==0.5.0
numpy
tensorflow==1.9.0
```

for installation of `keras-contrib` see [their repo](https://github.com/keras-team/keras-contrib)

## updates and todo

this is a complete rewrite using ATIS dataset, see the `old_version` branch for the original version

currently working on using the 2017 benchmark from snips:  
https://github.com/snipsco/nlu-benchmark

## files

- `00_data_processing-test.ipynb` for processing ATIS data
- `01_keras_modeling-test.ipynb` for training model
- `02_keras_demo.ipynb` for decoding new sentences on trained model
- `snips_00_preprocessing.ipynb` for processing SNIPS data

## background

A conversational spoken dialog system (SDS) traditionally consists of a pipeline of multiple elements operating in sequence; a common pipeline consists of *automatic speech recognition* (ASR) or *text-to-speech* (TTS), *natural language understanding* (NLU), a *dialog management system* (DMS) or *state tracking* module,*natural language generation* (NLG) and *text-to-speech* (TTS). *Natural Language Understanding*, also referred to as *Spoken Language Understanding* (SLU) when used as a component in an SDS, can be considered the act of converting a user's natural language request to a compact format that that the dialog manager can use to decide the proper response.

For *task-oriented* conversational agents ("chatbots"), the slot-filling paradigm is frequently used as seen in Lui & Lane 2016 and Goo *et al.* 2018, among others. This approach breaks down the NLU task into two primary sub-tasks, *slot-filling* (SF) and *intent detection\/determination* (ID). 

In this approach, a user's preferences are assigned to a number of **slots**, which the dialog manager attempts to fill with the **value** of the user's preference (in a programming analogy, a slot is a variable and the value is a variable's assigned value). These slots can be completely open, restricted to a set of types (such as days of the week, restaurant types, flight classes such as *first class* or *economy*), or boolean in value. T

Along with slots, the NLU must also classify **intent**. Intent captures the goal or purpose of a user's utterance. It can be roughly thought of as analogous to a programming *function* that takes a number of slots as arguments. Together, the intent and slots provide a summary of the user's utterance that can be used to determine how the chatbot responds.

The slot-filling and intent detection tasks can be handled separately, for example using a sequence model such as a Markov Model or conditional random field (CRF) for slot filling word-by-word, and a SVM or logistic regression model over the entire utterance for intent classification. However, with the growth of deep neural networks, "Joint NLU" has become a popular approach, in which a single network detects both intent and slot values. This allows for a simpler system (as it uses a single machine-learned model for both tasks), and allows for varying degrees of synergy in training the model. In some cases this may just mean using a single input for both tasks, but some networks may use the predictions of one task to assist with the other.

## purpose

This model demonstrates an example of a joint NLU model, borrowing concepts from a number of current papers on NLU and the related named entity recognition task. 

## dataset

the ATIS dataset can be obtained here:  
https://github.com/yvchen/JointSLU

## results

our intent accuracy is **96.3%**, which beats out Goo *et al.* 2018's reported score of 94.1%. 

our slot F1 score of 93.71 using the phrase-based `conlleval` approach, substantially lower than their 95.2. of course we can tune a lot of things here or try the greedy LSTM.

note: the `conlleval` python script is not included, see **licensing** for link to source file.

```
# TRAIN RESULTS

INTENT F1 :   0.9958062755602081  (weighted)
INTENT ACC:   0.9966502903081733
SLOT   F1 :   0.9943244981289089  (weighted, labels only)

# TEST RESULTS

INTENT F1 :   0.9573853410886499  (weighted)
INTENT ACC:   0.9630459126539753
SLOT   F1 :   0.9508826933073504  (weighted, labels only)
CONLLEVAL :   93.71
```

## examples

we can use the decoding script to load the saved model and weights, and so some decoding:

```
query: looking for direct flights from Chicago to LAX
slots:
{'connect': 'direct', 'fromloc.city_name': 'chicago', 'toloc.city_name': 'lax'}
intent: atis_flight

query: give me flights and fares from New York to Dallas
slots:
{'fromloc.city_name': 'new york', 'toloc.city_name': 'dallas'}
intent: atis_flight#atis_airfare

query: i want a first class flight to los angeles
slots:
{'class_type': 'first class', 'toloc.city_name': 'los angeles'}
intent: atis_flight
```

## references

Goo *et al* (2018): *Slot-Gated Modeling for Joint Slot Filling and Intent Prediction*  
NAACL-HCT 2018, available: http://aclweb.org/anthology/N18-2118

Hakkani-Tur *et al* (2016): *Multi-Domain Joint Semantic Frame Parsing using Bi-directional RNN-LSTM*  
available: https://www.csie.ntu.edu.tw/~yvchen/doc/IS16_MultiJoint.pdf

Kim *et al* (2015): *Character-Aware Neural Language Models*  
available: https://arxiv.org/pdf/1508.06615.pdf

Liu & Lane (2016): *Attention-Based Recurrent Neural Network Models for Joint Intent Detection and Slot Filling*  
INTERSPEECH 2016, available: https://pdfs.semanticscholar.org/84a9/bc5294dded8d597c9d1c958fe21e4614ff8f.pdf

Ma & Hovy (2016): *End-to-end Sequence Labeling via Bi-directional LSTM-CNNs-CRF*  
available: https://arxiv.org/pdf/1603.01354.pdf

Park & Song (2017): *음절 기반의 CNN 를 이용한 개체명 인식 Named Entity Recognition using CNN for Korean syllabic character*  
available: https://www.dbpia.co.kr/Journal/ArticleDetail/NODE07017625 (Korean)

Srivastava, R. K., Greff, K., & Schmidhuber, J. (2015). *Highway networks*.  
available: https://arxiv.org/pdf/1505.00387.pdf

## licensing

models may use layers from [keras-contrib](https://github.com/keras-team/keras-contrib) released under the MIT License

models may use layers from [keras utilities](https://github.com/cbaziotis/keras-utilities) by cbaziotis released under the MIT License

models may use layers written *with reference to* [keras2-highway-network.py](https://gist.github.com/iskandr/a874e4cf358697037d14a17020304535) by iskandr

phrase-based slot F1 score may be determined using a modified [conlleval python port](https://github.com/sighsmile/conlleval) by sighsmile (*not included due to lack of license*) or using Goo *et al.*'s `conlleval` [based script](https://github.com/MiuLab/SlotGated-SLU) in `utils.py` 

