---
title: "A Statistical Look at Greenberg's Universals"
author: "Sean Liu"
date: 2022-08-12T03:13:37-07:00
categories: ['english', 'linguistics', 'greenberg']
draft: true
---
# Introduction 

*Language* is a fickle thing. On one hand, it's "easy": presumably, you and all the people you know speak at least one language. Many people speak more than one! But on the other, it's incredibly intricate and complex: even within a language, the choice of words can subtly influence the tone and flow of a sentence, not to mention all the nooks and crannies of varying dialects. Across languages and language families, this difference seems to multiply thousandsfold: in some languages, "I eat an apple" becomes (when literally translated) "I apple eat," each with their own set of sounds and ways to form sentences. Therefore, it's a natural question to ask: what do languages have in common? Specifically, what do *human* languages have in common that are not also shared by *animal* calls and grunts? 

This leads to the Chomskian theory of Universal Grammar, which postulates that each and every human is gifted with an innate abstract ability for language, from which all language all stem from this central source - realisations of this invisible Form. More concretely, it has been proposed that *recursion*, the ability to embed parts of language inside itself, is the unifying and differentiating factor between human languages and animal ones (though even this is disputed: see Daniel Everett's work on Pirahã linguistics). 

Such *Universals* (facts that are true across all or most languages) are thus invaluable, because they potentially hold clues to the inner workings of our linguistic faculties. Here, we aim to introduce Greenberg's Universals and to perform a statistical verification of the proposed Universals.

# What are Greenberg's Universals? 

Let's begin with a bit of history. In 1963, American linguist Joseph Greenberg published an article named *Some Universals of Grammar with Particular Reference to the Order of Meaningful Elements*. In the paper, he proposed a number of universals from an analysis of 30 languages, covering a wide distribution of the world's languages. He notes that, 

> This sample was selected largely for convenience. In general, it contains languages with which I had some previous acquaintance or for which a reasonably adequate grammar
was available to me. Its biases are obvious, although an attempt was made to obtain as
wide a genetic and areal coverage as possible. 

The types of universals were also slightly different from earlier, as some of them were implicational ("If language A has feature *X*, then it also has feature *Y*"). In addition, he notes that the condition does not go both ways (i.e. if *X* is *not* found, then we can say nothing about *Y*). The universals include both syntactic and morphological propositions: nothing much is said about other aspects of language, though this does not mean that there no such universals are there. To give a flavour of the sorts of universals that have been proposed, here are some samples: 

* Universal 1. In declarative sentences with nominal subject and object, the dominant order
is almost always one in which the subject precedes the object.

For example, consider the sentence *I ate the apple*. Its subject (*I*) is far more likely to appear before the object (*apple*). Even in other languages such as Japanese, we have 

*僕 は りんご を 食べ-た*
I SUBJ apple direct-object-marker Eat-past

* Universal 36. If a language has the category of gender, it always has the category of
number. 

For example, Latin has gender categories: *unus fluvius* (river, masculine), *una lettera* (letter, feminine), and also number: *unus fluvius* (one river), *multi fluvii* (many rivers). 


In total, 45 universals were proposed. 
# The World Atlas of Language Structures Database

Whereas Greenberg only considered 30 languages, in the sixty years since the paper's publication, much work has been done on collecting data from the world's languages, and the [World Atlas of Language Structures](https://wals.info/) (WALS) database is a compilation of many aspects of thousands of languages, sourced from a multitude of studies. It is then possible to leverage this new data source and try to verify statistically some of the claims that were made over half a century ago. We will verify claims primarily on the availability of data: nine features were collected, each with 

