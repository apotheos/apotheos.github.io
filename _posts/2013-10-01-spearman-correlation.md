---
layout: post
title: "Identifying Languages with the Spearman Correlation"
description: "A Journey into Natural Language Processing Using NLTK"
category: articles
tags: ['NLP', 'CSC']
comments: true
# permalink: "/path/to/permalink.html"
# published: true
---

The problem below is taken from Chapter 3 of *[Natural Language Processing with Python](http://nltk.org/book)*. 

> 43\. With the help of a multilingual corpus such as the Universal Declaration of Human Rights Corpus
  (`nltk.corpus.udhr`), along with NLTK's frequency distribution and rank correlation functionality
  (`nltk.FreqDist`, `nltk.spearman_correlation`), develop a system that guesses the language of a
  previously unseen text. For simplicity, work with a single character encoding and just a few languages.

See [CSC499-NLP/ch_3/exercises/languages_guessing.py](https://github.com/apotheos/CSC499-NLP/blob/master/ch_3/exercises/language_guessing.py) for the solution.

So, with the problem in mind I decided to make it harder for myself by ignoring the authors' suggestions in the last
sentence above. Namely, I work with Latin-1 and UTF-8, which allows me to use a large percentage of the UDHR corpus.

Understanding the Algorithm
---------------------------

First I need to figure out what the Spearman correlation is. From Wikipedia:

> In statistics, Spearman's rank correlation coefficient or Spearman's rho, named after Charles Spearman and often denoted by the Greek letter \rho (rho) or as r_s, is a nonparametric measure of statistical dependence between two variables. It assesses how well the relationship between two variables can be described using a monotonic function. If there are no repeated data values, a perfect Spearman correlation of +1 or −1 occurs when each of the variables is a perfect monotone function of the other.

And from NLTK's documentation on the `spearman_correlation(rank1, rank2)` function:

> Returns the Spearman correlation coefficient for two rankings, which
  should be dicts or sequences of (key, rank). The coefficient ranges from
  -1.0 (ranks are opposite) to 1.0 (ranks are identical), and is only
  calculated for keys in both rankings (for meaningful results, remove keys
  present in only one list before ranking).

From these bits of information it is clear that the Spearman correlation takes two ordered rankings calculates how
different they are from one another. A ranking is basically a frequency distribution with the "count" replaced with a rank
showing how often a word occurs in a text relative to the other words. This means I can store each training set and the
sample data as `FreqDist`s right until I actually use the `spearman_correlation(r1, r2)` function.

I also noted the last parenthetical of the NLTK documentation: "for meaningful results, remove keys present in only one
list before ranking".

This means that given the following `FreqDist`s `a` and `b` given below, I could only get "meaningful results" by
doing the following:

~~~ python
>>> # Assume `a` and `b` are of type FreqDist and have already been hydrated with data.
>>> dict(a)
{
  "dog": 4358,
  "cat": 424,
  "reptile": 48
}
>>> dict(b)
{
  "cow": 334,
  "cat": 44,
  "zebra": 2
}
>>> keys_in_common = set(a.keys()) & set(b.keys())
>>> # Remove the keys from a that are not in b
>>> for k in set(a.keys()) - keys_in_common:
...     del a[k]
...
>>> # Remove the keys from b that are not in a
>>> for k in set(b.keys()) - keys_in_common:
...     del b[k]
...
>>> # Convert FreqDists to rankings.
>>> spearman_correlation(ranks_from_sequence(a.keys()), ranks_from_sequence(b.keys()))
~~~

I'll revisit how this all worked later, though.

Making the Training Set(s)
--------------------------

First I need to create FreqDists for each text in the UDHR Corpus and place them in a map of {"language": FreqDist}.
This bit of preprocessing allows me to easily calculate and compare ranks for sample texts of unknown languages. I can
accomplish this by looping through each file id (each file is a different language), and making a frequency distribution
out of it:

~~~ python
result[language] = FreqDist(udhr.words(_id))
~~~

In the above code, assume `_id` has been previously initialized with one of the file names in the UDHR corpus,
`language` with the a string representing the language the file contains, and `result` is a dictionary that maps
language strings to `FreqDist`s. After looping through all files with the encodings I specified earlier, I am done, and
ready to tackle the algorithm portion.

Solving the Problem
-------------------

Since most of the code for this is above, let's just look at the final product:

~~~ python
def predict_language(sample_fd, training_set_fds):
    scores = dict()

    for language, language_fd in training_set_fds.iteritems():
        # make copies so we don't alter the originals
        sfd = dict(sample_fd)
        lfd = dict(language_fd)

        # make sure both frequency distributions have only the keys they have in common
        delete_differences(sfd, lfd)

        scores[language] = spearman_correlation(
            ranks_from_sequence(sfd),
            ranks_from_sequence(lfd)
        )

    return sorted(scores.items(), key=lambda x: x[-1], reverse=True)
~~~

Using the above algorithm on the text of *Moby Dick*, I got the results below:

~~~
Punjabi_Panjabi.................1.0
Japanese_Nihongo................1.0
Mongolian_Khalkha...............0.8
Sinhala.........................0.572727272727
Nepali..........................0.4
Hindi...........................0.4
Farsi_Persian...................0.396428571429
Hmong_Miao......................0.371428571429
Chickasaw.......................0.370588235294
Mapudungun_Mapuzgun.............0.351464693185
Luvale..........................0.319648093842
Hindi_web.......................0.3
Estonian_Eesti..................0.291133238502
Rukonzo_Konjo...................0.290306122449
Esperanto.......................0.289177489177
Kapampangan.....................0.285067873303
Siswati.........................0.279838709677
Waray...........................0.278019674246
Basque_Euskara..................0.276197249881
Ditammari.......................0.274424663482
Asante..........................0.270634920635
Latvian.........................0.269559032717
Dangme..........................0.267695099819
Sammarinese.....................0.258786049631
Hebrew_Ivrit....................0.25
Tzeltal.........................0.243286474515
Malagasy........................0.236890645586
Runyankore......................0.233709483794
Kituba..........................0.231951871658
Tzotzil.........................0.231614823349
Asturian_Bable..................0.229883993583
Tahitian........................0.222001527884
Swahili_Kiswahili...............0.220024161885
Swaheli.........................0.220024161885
Balinese........................0.218824087245
Kasem...........................0.215909440463
Hmong_Miao_Northern.............0.214115031953
Kinyarwanda.....................0.212334258403
Galician_Galego.................0.207357588526
Rarotongan_MaoriCookIslands.....0.205355884426
HaitianCreole_Popular...........0.204761904762
Kazakh..........................0.203302373581
Russian_Russky..................0.203007518797
Russian.........................0.203007518797
Sanskrit........................0.2
Bulgarian_Balgarski.............0.19649122807
Rundi_Kirundi...................0.195388669302
Xhosa...........................0.194117647059
Norwegian_Norsk.................0.191215034965
HaitianCreole_Kreyol............0.187767456424
Bosnian_Bosanski................0.186699507389
Frisian.........................0.18596390169
Sardinian.......................0.185131430414
Uzbek...........................0.183108633653
Swedish_Svenska.................0.17956254272
Spanish_Espanol.................0.179459346624
Spanish.........................0.179459346624
Ibibio_Efik.....................0.171428571429
Afrikaans.......................0.169022022753
Achuar..........................0.168547008547
Romani..........................0.163780663781
Luba............................0.160452961672
Chechewa_Nyanja.................0.158310206599
Nyanja_Chechewa.................0.158310206599
Rhaeto..........................0.158185683912
Aymara..........................0.157203261854
Portuguese_Portugues............0.154809907834
Soninke_Soninkanxaane...........0.153834322803
Vlach...........................0.152194606029
Dagbani.........................0.150772645778
Hungarian_Magyar................0.150469043152
Oshiwambo_Ndonga................0.14835800185
Uighur_Uyghur...................0.148257839721
Picard..........................0.147421328671
Adja............................0.144079555967
Yao.............................0.143434343434
Zulu............................0.143328445748
Krio............................0.140588540316
Sukuma..........................0.137022397892
~~~

That's weird... I wonder why that happened...

I need to reconsider what I'm doing. When I rank only the keys the two `FreqDist`s have in common, what I'm doing
is comparing the occurences of 'the' in *Moby Dick* to the occurrences of 'the' only in languages that have the word
'the'. That doesn't make very much sense algorithmically, since we should be comparing the whole range of words in each
language. That is, we should not be eliminating keys that are not set of keys the two sets have in common.

When I comment out the line that does this:

~~~ python
# make sure both frequency distributions have only the keys they have in common
# delete_differences(sfd, lfd)
~~~

And run it again, I get something that seems more reasonable:

~~~
English.........................-3814.33128572
NigerianPidginEnglish...........-11306.2529306
Interlingua.....................-65529.1688999
French_Francais.................-86099.9348059
Friulian_Friulano...............-86695.7041187
Seereer.........................-89096.0581801
Catalan.........................-100188.142584
OccitanLanguedocien.............-100426.612906
Rhaeto..........................-101072.480227
Zapoteco........................-102046.914637
Yapese..........................-102050.06669
Catalan_Catala..................-103555.181202
Asturian_Bable..................-105620.301046
Breton..........................-106172.853931
Walloon_Wallon..................-113286.146384
Sammarinese.....................-115414.038431
Ido.............................-118947.886309
Chinanteco......................-119250.502404
SolomonsPidgin_Pijin............-119426.272847
Paez............................-120131.453011
OccitanAuvergnat................-123346.087319
Kiche_Quiche....................-126368.5195
Luxembourgish_Letzebuergeusch...-127480.10229
Spanish_Espanol.................-131259.233358
Spanish.........................-131259.233358
Moore_More......................-131925.45684
TokPisin........................-132372.363336
Mam.............................-132750.885857
HaitianCreole_Kreyol............-136437.167937
Beti............................-137193.089332
Chamorro........................-140047.209989
Wolof...........................-140399.17693
Picard..........................-141945.553846
Tzotzil.........................-142950.844734
Bichelamar......................-143851.972369
Qechi_Kekchi....................-147748.57893
Danish_Dansk....................-147949.934477
Mayan_Yucateco..................-148387.515232
Norwegian.......................-148969.642535
Sussu_Soussou...................-151539.158067
Afrikaans.......................-151650.043816
Jola............................-153494.019802
Fon.............................-154263.860433
Tetum...........................-158596.596172
Kurdish.........................-159765.008212
Vlach...........................-160530.736727
Mazahua_Jnatrjo.................-160933.92329
Norwegian_Norsk.................-160965.573033
Tiv.............................-162065.812908
Peuhl...........................-164046.900273
Pulaar..........................-164046.900273
Tzeltal.........................-165460.903442
Otomi_Nahnu.....................-165981.639451
Huasteco........................-167169.392332
Batonu_Bariba...................-169328.082622
Faroese.........................-171169.397827
Gonja...........................-174190.777256
Marshallese.....................-175794.31945
Guen_Mina.......................-176230.337927
ScottishGaelic_GaidhligAlbanach.-176480.065634
Soninke_Soninkanxaane...........-178340.499985
Romani..........................-178518.491306
Dagaare.........................-178714.939027
Italian.........................-181099.33159
Italian_Italiano................-181099.33159
Portuguese_Portugues............-181257.067684
Lamnso_Lam......................-181851.054164
Frisian.........................-182641.207349
Makonde.........................-183614.634977
Esperanto.......................-184276.807612
Yao.............................-184998.276234
Mixteco.........................-185652.815376
Trukese_Chuuk...................-185660.318206
Chuuk_Trukese...................-185660.318206
HaitianCreole_Popular...........-190173.695094
Baoule..........................-190589.807038
IrishGaelic_Gaeilge.............-190848.048013
Achehnese.......................-191414.41039
Palauan.........................-191850.327425
Maltese.........................-192288.187308
Igbo............................-193027.005932
Ponapean........................-194019.071334
Wayuu...........................-194506.045718
NorthernSotho_Pedi..............-196612.645124
Lozi............................-197011.511458
Tenek_Huasteco..................-199337.428589
Mazateco........................-199460.811706
Uzbek...........................-201266.440789
Tojol...........................-202902.477706
Luganda_Ganda...................-203066.125973
Swedish_Svenska.................-204774.998291
Kasem...........................-207138.808176
Corsican........................-208068.446984
Dangme..........................-208826.629634
Wama............................-208880.511003
Welsh_Cymraeg...................-208881.764292
Amarakaeri......................-209392.593888
Runyankore......................-209565.957167
Hmong_Miao_Northern.............-210096.123034
Maninka.........................-215073.61732
Ewe_Eve.........................-218439.721746
Asante..........................-219884.146284
Kpelewo.........................-223564.297623
Mikmaq_Micmac...................-224194.013937
Albanian_Shqip..................-227327.302868
Cakchiquel......................-228126.717791
Dutch_Nederlands................-230487.57904
Ditammari.......................-233288.846559
Kapampangan.....................-235100.536028
Chayahuita......................-237516.510108
Miskito_Miskito.................-239085.941276
Cebuano.........................-239552.894525
Adja............................-240527.07389
~~~

This makes much more sense as a result, but the numbers returned by the `spearman_correlation(...)` are way outside of
the function's supposed range. However, it is clear to me that this result *is* accurate, especially since English,
Pidgin English, Interlingua, and French are all very close to one another, and that is represented in the results.
Trying this algorithm with different samples produces similarly correct results. I would love to know exactly why
this works, but only have a general idea at the moment.
