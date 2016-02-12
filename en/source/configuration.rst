====================
Change Configuration
====================

Do you remember that you set the ``-f`` option when starting ``jubaclassifier``?
At the point, you also have change a lot of other setting parameters, which are very important for specifying used algorithms and performance tuning.
Next we would like to introduce how to change it.


See configuration
=================

Let's look into the configuration file that we used in the previous section, ``pa1.json``.

::

   $ cat /opt/jubatus/share/jubatus/example/config/classifier/pa1.json
   {
     "converter" : {
       "string_filter_types" : {},
       "string_filter_rules" : [],
       "num_filter_types" : {},
       "num_filter_rules" : [],
       "string_types" : {},
       "string_rules" : [
         { "key" : "*", "type" : "str", "sample_weight" : "bin", "global_weight" : "bin" }
       ],
       "num_types" : {},
       "num_rules" : [
         { "key" : "*", "type" : "num" }
       ]
     },
     "parameter" : {
       "regularization_weight" : 1.0
     },
     "method" : "PA1"
   }


Hard to get it at first sight, isn't it?
We use a type of JSON file for each server program in Jubatus.
Every setting to declare how the machine learning algorithm works is defined inside the JSON file.
On the top level, the configuration file has three major components.

.. csv-table::
   :header: "Field name" "Explanation"

   converter, Defines how to pre-process input data
   parameter, Specifies the parameters for machine learning algorithm
   method, Indicates which algorithm to use

We will explain each of them by changing the values, from bottom to top.
Note that the details are available on the `Web <http://jubat.us/ja/api_classifier.html>`_ .


Change algorithm
================

Every machine learning algorithm is responsible for building a prediction model and determining how to update the model when receiving training samples.
Jubatus is specialized for a type of machine learning algorithm named *online learning*, of which aim is to update the model each time when receiving a training sample one-by-one.

In the configuration file, ``method`` component defines which algorithm to use.
For trying to change it, copy the sample configuration file to an arbitrary directory.
First we change the value of ``method`` from "PA1" to "AROW".

::

   $ cp /opt/jubatus/share/jubatus/example/config/classifier/pa1.json ./my_conf.json
   $ vi my_conf.json
   $ jubaclassifier -f my_conf.json

Then run ``gender.pyy``. The output must be slightly different.

"PA1" stands for the Passive-Aggressive algorithm, proposed in 2003.
"AROW" is a newer algorithm, "Adaptive Regularization of Weight Vectors", proposed in 2009.
We have seen substantial progress each year in machine learning algorithms for decades.
Though "PA1" and "AROW" are both for classification problems, "AROW" contains improvements in learning efficiency and robustness against noisy data.
On the other hand, "PA1" is faster than "AROW" given the same amount of training samples, as better algorithm tends to be more complex.


Jubatus is also equipped with other famous classification algorithms such as "NHERD" and "CW", though we refer interested users to the documents on the Web.
It would be nice to try them and to evaluate how the performance is difference between the algorithms.
Note that the algorithm cannot be changed during running ``jubaclassifier``.


Change parameter
================

Machine learning algorithm corresponds to a prediction model having a large number of model parameters, and they will be controlled based on given training data.
On the other hand, there are also a few configuration parameters that are specified in the configuration file in advance of training.
They control the each process in the training phase, for example, how aggressively the algorithms update the current model for each time.
These parameters are often called as *hyper-parameters* since they are unchanged by training, unlike the model parameters.

Hyper-parameters can have many roles in general.
Those of ``jubaclassifier`` roughly represent the sensitivity of algorithm to a training sample.
Higher sensitivity makes the model training faster but less robust to noisy sample.
Conversely, lower sensitivity leads to slower and more robust training.
The trade-off between them cannot be tuned in advance, so it used to be empirically selected using real samples by changing the hyper-parameters.

OK, then let's do that by yourself.
First we modify the value of ``regularization_weight`` and run the example.

::

   ...
   "parameter" : {
     "regularization_weight" : 10.0
   },
   ...


It is necessary to increase or decrease the value of ``regularization_weight`` by order of 10 to search for the best value.
The current version of Jubatus does not have any capability to automatically choose the best value by model selection techniques.

Change feature value
====================

The remaining part in configuration is ``converter``, which is the most important.
We would like to describe it in more detail since the success of predictive analytics using Jubatus strongly depends on the converter of feature values.

Machine learning algorithms cannot directly deal with raw data in the real-world application, such as documents in text analytics, nor images in computer vision.
Instead, they only uses a set of feature values, which are often represented as numerical feature vectors of fixed size.
So, how to apply machine learning techniques to complex raw data in such applications?
Feature extraction (or feature convert) is the intermediate process, to transform raw data into feature vectors.
Depending on the raw input data and the machine learning algorithm, a pre-defined format of feature vectors will be extracted and used in the following training phase.
Since most of the machine learning algorithms assume vectors as input, they can work given them, no matter what is the original raw data, documents or pictures.

Machine learning libraries tend to be lack of this feature extraction module.
Therefore, it is up to users to write the feature value costruction logic as pre-processing before sending them to the libraries.
In contrast, Jubatus has a built-in feature value converter on the server side, so that users can directly input raw data and run machine learning algorithms in Jubatus.


Default configuration
---------------------

For making it easier to understand how to change, we will explain what kind of configuration is used in ``jubaclassifier``, by going through the default configuration settings.


Speaking of input data to Jubatus, though Jubatus is originally designed for handling any kinds of raw unstructured data, currently only three types, symbol sequences such as strings, numeric values such as sensor data, and binary values such as images, are supported.
They will be separately processed in the following feature value extraction.

In this example, we will see feature extractions for string and numerical values.
For instance, let us assume that the following information is given as input.

::

   {
     "hair": "short",
     "top": "T shirt",
     "bottom": "jeans",
     "height": 1.70
   }


Now, we have to convert it into a numerical vector.
It used to be represented as a sequence of numerical values, such as (1.5, 2.3, 4.2), but we use a set of pairs of dimension keys and values.
There would be many other dimensions with zero values, but we assume that they are omitted in this format.

A result of simple vectorization is as follows.

::

   {
     "hair=short": 1.0,
     "top=T shirt": 1.0,
     "bottom=jeans": 1.0,
     "height": 1.70
   }

You can see that different processes have been applied to strings and values. 

The process to strings can be regarded as a transformation from a nominal value to a value of dummy variable.
This rule is defined inside ``string_rules`` in the configuration.
Let's look at the default setting for it.

::

   ...
       "string_rules" : [
         { "key" : "*", "type" : "str", "sample_weight" : "bin", "global_weight" : "bin" }
       ],
   ...

These three lines just indicates the following four settings.

1. 'key: "*"' means this rule is applied to all of the input variable with any variable IDs (keys).
2. 'type: "str"' means each of the variables will be dealt with as string-type and correspond to one dimension in feature vector.
3. 'sample_weight: "bin"' means each value in feature vector will have binary value (1.0).
4. 'global_weight: "bin"' also means the global weight for these feature values will be 1.0.

This rule generally shows how to extract feature values using a method specified with "type" from a subset of input variables that are matched with "key".
"sample_weight" means that the feature value will be 1.0 if the "type" method matches at least once in the input variable, and 0.0 otherwise.
The final feature values will be the multiplication of "sample_weight" and "global_weight", so in this case, 1.0 or 0.0.

Let's look at how to handle numerical information such as height.
The configuration has ``num_rules`` component as follows.

::

  ...
    "num_rules" : [
      { "key" : "*", "type" : "num" }
    ]
  ...

This rule is much simpler.

1. 'key: "*"' means this rule is applied to all of the input variable with any variable IDs (keys).
2. 'type: "num"' means each of the variables will be dealt with as numerical-type and its raw value will be used in feature vector.

The input value 1.70 will be contained in the feature vector as-is.

By modifying these rules, feature vectors can seize many different aspects of raw input variables.


Tricks in feature extraction
----------------------------

We will show an example of how to extract a value that you interested in as feature value.
The following is just a pair of name and address of a person.
::

   {
     "name": "David Johnson",
     "address": "GatewayPlace SanJose CA"
   }

For the analysis point of view, this representation of address has too small granularity to compute some statistics.
Instead, you might want to make groups of living people in level of "SanJose" or "CA".
In other words, the following lines look exactly the needed information.

::

  {
    "name=David Johnson": 1.0,
    "address=GatewayPlace" : 1.0,
    "address=SanJose": 1.0,
    "address=CA": 1.0
  }

Then let's try to extract indivisual terms in ``address`` by separating with whitespace.
As meantioned earlier, we can use ``string_rules`` to specify such rule.
So we append a new subrule for whitespace separation in it, named ``space`` as follows.

::

   ...
       "string_rules" : [
         { "key" : "name", "type" : "str",
           "sample_weight" : "bin", "global_weight" : "bin" },
         { "key" : "address", "type" : "space",
           "sample_weight" : "bin", "global_weight" : "bin" }
       ],
   ...

Note that new subrule has the type name ``space`` instead of ``str``.
Then, just run it.
In the previous output you can see that David as a person living in "Gateway Place, San Jose, CA".
Now David is a person who lives "Gateway Place", "San Jose", and "CA".
This really helps how the following learning task works depending on arbitrary abstraction level of the address.

In a similar manner, you can regard the feature extraction method as an control of abstraction levels on the raw input.
More precise the abstraction level of feature vector becomes, trained model can be more sensitive to small difference but also it requires more training data to learn.
In contrast, less precise the abstraction level is, trained model can be easily learned, but it cannot distinguish small difference in the raw input.
That means there is a trade-off.
Though it depends on the application or data, for example, applications with text documents can empirically go well with the level of word (not the characters), as we did above.
We omit the more complex examples using the co-occurrence of words in this tutorial.


Use plug-ins
------------

In addition to the example for whitespace segmentation, we would like to show another configuration.
Note that others can be found on the official Web document.

Given a natural language text, you cannot extract good features with whitespace segmentation.
Instead, we use natural language processing, which is a built-in capability of Jubatus.
Morphological analysis, which breaks a sentence into a sequence of words, used to be the most important part, 
and Jubatus is equipped with MeCab, a widely used open source software for it.


The configuration rule is slightly more complex.
First, you have to use MeCab as a plug-in, which can be loaded as follows.
``string_types`` shows how to load the plug-in, and ``string_rules`` defines how to apply it to input data.

::

   ...
   "string_types": {
   "mecab": {
       "method": "dynamic",
       "path": "libmecab_splitter.so",
       "function": "create",
     }
   },
   "string_rules" : [
     { "key" : "*", "type" : "mecab", "sample_weight" : "bin", "global_weight" : "bin" }
   ],
   ...


You can also build your own plug-in as an external library like this. Let's try if you are interested in other time.


For numerical data and others
-----------------------------

Configuration for handling numerical values is also very important for performance tuning.
It is also most the same for numerical input data to change configurations by replacing ``string`` with ``num``.
Please refer to the official Web.

Other capabilities of configuration include filtering, to remove unnecessary information from the raw data such as tags in HTML documents, or redundant characters in certain types of templates.
