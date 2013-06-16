============
Introduction
============

In this section we briefly introduce machine learning and Jubatus.
Please refer to the references if you need more detailed studies.


Machine learning
================

Machine learning is a generic term given to a set of computational algorithms for computers to make decisions based on data.
The key is that it is all depending on the data.
It also closely relates to other technical fields such as statistics.

Before machine learning becomes popular, adaptive decisions by computers were made according to a set of rules, which define when and what to do.
Rule-based decision is very useful since it is easy to implement and understand how and why a decision was made.
For more complex and ambiguous decisions, however, it is hard to derive and maintain a large number of combinations of rules required to find an answer by taking into account many conditions and exceptions.
On the other hand, machine learning algorithms generally try to find a model, which corresponds to a selected combinations of rules that are sufficient to fix the characteristics of a given dataset.
Especially, when computers can use a set of different data sources, it is more reasonable and robust to automatically make a decision based only on the data, rather than relying on manual rules.
Thanks to the progress of algorithms and their practical performance, application of machine learning is expanding in many fields. It includes, for example, advertisement optimization based on the click-through history of candidate banner ads, spam filtering based on the past spam e-mails, recommendation in e-commerce based on his/her past purchase behaviors, or fraud detection in credit card usages.

Machine learning involves a lot of problem settings.
In this tutorial we focus on the classification problem, which is one of the most commonly-used machine learning tasks in practice.
The task is to predict a class, for example, YES or NO to each input.
In the training phase, a pair of a data sample and an answer is given to the algorithm.
The algorithm can learn their relationship and tendency on when a sample belongs to a specific class.
After repeating the process for a set of training samples, the obtained model can predict the class, YES or NO, to unseen test samples.

.. figure:: ml.png
   :width: 800px

   Comparison between machine learning and human's learning

For example, Bayesian filter, which is known as a basis for spam filtering, is also one of the classification problems.


Since classification is very simple, there are many applications.
The above example with two classes YES and NO is called binary classification problem.
"Spam or ham?", "Male or female?", "Clicked or ignored?", these decisions can be all recognized as a sort of binary classification problem.

When having more than two classes, it is called multiclass classification problem.
For example, the task belongs to it to wacth a picture of dishes and predict them as french, italian, or chinese.
The classifier in Jubatus can address the multiclass classification problems, which is a generalization of binary case.
The purpose of this tutorial is to learn how to use the Jubatus' classifier (jubaclassifier).


What is Jubatus?
================

Jubatus is an open source software platform for machine learning, which were originally developed by NTT SIC and Preferred Infrastructure, Inc.
Not only classification, it also supports other machine learning tasks such as regression, recommendation, and anomaly detection.

The main advantage is that Jubatus can work in a distributed environment.
We often have a large amount of data for machine learning problems.
Jubatus is designed to make the computation distributed among multiple servers so that it can ahieve scale-out for higher throughput by adding more servers.
Though this tutorial do not cover the distributed running, please see and try the examples on the official Web or github.com if you are interested.

The another feature of Jubatus is that it focuses on *online learning* in machine learning algorithms.
Typically, training of machine learning models started after all of the traning samples are collected and stored in a place (batch learning).
Recently, on the other hand, many efforts have beed done for online learning, which is to receive and process each traning sample without storing them.
That's the difference between batch and online learning algorithms.
Though special case for online learning is not required in this tutorial, you might realize its advantage in the future.

.. note::

   In general, online learning tends to be worse than batch learning in terms of prediction accurcy since it can only see a part of the samples at the same time. However, thanks to the research progress on online learning algorithms, the difference is becoming smaller especially when we have enough amout of samples.
