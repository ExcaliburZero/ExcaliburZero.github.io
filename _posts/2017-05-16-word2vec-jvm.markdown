---
layout: post
type: article
title:  "A Quick Tutorial on using pre-trained Word2Vec vectors on the JVM"
author: "Christopher Wells"
image: "/images/word2vec_jvm.png"
date:   2017-05-19 09:10:00
categories: word2vec jvm
---
Recently I wanted to use Word2Vec in a project I was working on in order to do some quick word similarity comparisons. I was able to find a few JVM libraries for wokring with Word2Vec, but most of their documentation focused on how to train Word2Vec models, rather than how to docompairisons using pre-trained models.

After a while I was able to figure out how to do some word similarity compairisons with a pre-trained model, but I was annoyed that there weren't too many resources out there on how to do this. So I decided to put together this tutorial to make the process easier for others who want use Word2Vec in this way.

In this tutorial, I will walk you through the process including getting a pre-trained word vectors file and using in a Word2Vec library to do some simple word similarity comparisons.

## Finding a pre-trained vectors file
In order to use Word2Vec to do some word similarity comparisons, you must have a trained Word2Vec model. Most Word2Vec libraries allow you to create a model by either training a model from scratch or by providing a pre-trained model though a word vectors file.

In this tutorial we will be using a pre-trained model. There are a number of pre-trained Word2Vec models available on the internet.

Here are a few vectors files. I recommend starting out with the 55 MB one, as it will reduce the chances of running into the default JVM memory limit.

* 55 MB - [https://github.com/Refefer/word2vec-scala/blob/master/vectors.bin?raw=true](https://github.com/Refefer/word2vec-scala/blob/master/vectors.bin?raw=true)
* 1.5 GB - [https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit?usp=sharing](https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit?usp=sharing)

## Selecting a Word2Vec library
Next you will need to select the Word2Vec library that you will use. Luckily there are several different Word2Vec libraries for the JVM. A quick search on Maven Central reveals the following.

* [org.allenai.word2vec](https://search.maven.org/#artifactdetails%7Corg.allenai.word2vec%7Cword2vecjava_2.11%7C1.0.1%7Cjar)
* [com.medallia.word2vec](https://search.maven.org/#artifactdetails%7Ccom.medallia.word2vec%7CWord2VecJava%7C0.10.3%7Cjar)
* [org.deeplearning4j](https://search.maven.org/#artifactdetails%7Corg.deeplearning4j%7Cdeeplearning4j-scaleout-akka-word2vec%7C0.0.3.1%7Cjar)

For this tutorial I will show how to use the `org.allenai.word2vec` library, as that was the library I used for my project.

You can find how to add the library to your build dependencies in your build tool on the [Maven Central entry for the library](https://search.maven.org/#artifactdetails%7Corg.allenai.word2vec%7Cword2vecjava_2.11%7C1.0.1%7Cjar). If you are not yet using a build tool, I would reccomend using one as they make it much easier to be able to use external libraries.

In my case I was using SBT as my build tool, so I had to add the following line.

~~~~
libraryDependencies += "org.allenai.word2vec" % "word2vecjava_2.11" % "1.0.1"
~~~~

## Loading the vectors and comparing words
Once you have gotten your vectors file and added the library as a dependency, you can now begin using Word2Vec. I will show some quick code for doing word similarity compairisons using a pre-trained model. To keep things simple I will show the code in Scala, but the general ideas apply to most JVM languages.

~~~~
import java.io.File
import org.allenai.word2vec.{Searcher, Word2VecModel}

// Load the Word2Vec model from the vectors file
val vectorFile = new File("vectors.bin")
val searcher = Word2VecModel.fromBinFile(vectorFile).forSearch()

// Compare two words for similarity
val first = "grass"
val second = "field"
val diff = searcher.cosineDistance(first, second)

println(first + " ~ " + second + " => " + diff)
~~~~

First you must load in the vectors file as a Word2Vec model. This can be done by creating a file object for the vectors file and then passing it into the `fromBinFile` method of the `Word2VecModel` class. Additionally you will need to call the `forSearch` method on the resulting model in order to get a searcher that will allow you to compare words using the model.

It is a good idea to call the `forSearch` method once early on rather than each time you try to compare two words, as it does take a bit of time to run, and if you are comparing a lot of words then the time can add up.

~~~~
// Load the Word2Vec model from the vectors file
val vectorFile = new File("vectors.bin")
val searcher = Word2VecModel.fromBinFile(vectorFile).forSearch()
~~~~

Now that you have the Word2Vec searcher you can begin to compare words. You can use the `cosineDistance` method to get a similarity value for the two given words. Lower values tend to indicate words that are more similar, for example "grass" and "field" may yield 0.07 while "grass" and "fireworks" may yield 0.30.

~~~~
// Compare two words for similarity
val first = "grass"
val second = "field"
val diff = searcher.cosineDistance(first, second)

println(first + " ~ " + second + " => " + diff)
~~~~

Now if we run this code, we will get something like the following for the output. The similarity value may differ depending on the word vectors file that you use.

~~~~
scala> println(first + " ~ " + second + " => " + diff)
grass ~ field => 0.0655889737862099
~~~~

You should also be sure to before you do word compairisons, check that the words you are attmpting to compare are actually in the pre-trained model.

## Conclusion
In this tutorial I have shown how to quickly setup Word2Vec with a pre-trained vector file and compare words for similarity. If you are interested in doing more with Word2Vec I recommend checking out the [GitHub repo](https://github.com/allenai/Word2VecJava) for `org.allenai.word2vec`. It's documentation is okay and it's source code is fairly approachable.
