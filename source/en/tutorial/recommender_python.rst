Python
==================

Here we explain the sample program of Recommender in Python.

--------------------------------
Source_code
--------------------------------

In this sample program, we will explain 1) how to configure the learning-algorithms that used by Jubatus, with the example file 'npb_similar_player.json'; 2) how to train the model by 'Update.py' with the training data in 'baseball.csv' file, and how to recommend players with 'Analyze.py'. Here are the source codes of 'npb_similar_player.json', 'Update.py' and 'Analyze.py'.

**npb_similar_player.json**

.. code-block:: python

 01 : {
 02 :   "method": "inverted_index",
 03 :   "converter": {
 04 :     "string_filter_types": {},
 05 :     "string_filter_rules": [],
 06 :     "num_filter_types": {},
 07 :     "num_filter_rules": [],
 08 :     "string_types": {},
 09 :     "string_rules": [],
 10 :     "num_types": {},
 11 :     "num_rules": [
 12 :       {"key" : "*", "type" : "num"}
 13 :     ]
 14 :   },
 15 :   "parameter": {}
 16 : }


**Update.py**

.. code-block:: python

 01 : #!/usr/bin/env python
 02 : # -*- coding: utf-8 -*-
 03 : 
 04 : import sys, json
 05 : from jubatus.recommender import client
 06 : from jubatus.recommender import types
 07 : 
 08 : NAME = "recommender_baseball";
 09 : 
 10 : if __name__ == '__main__':
 11 :     # 1.  Connect to Jubatus Server
 12 :     recommender = client.recommender("127.0.0.1",9199)
 13 :     # 2. Prepare the training data
 14 :     for line in open('dat/baseball.csv'):
 15 :         pname, team, bave, games, pa, atbat, hit, homerun, runsbat, stolen, bob, hbp, strikeout, sacrifice, dp, slg, obp, ops, rc27, xr27 = line[:-1].split(',')
 16 :         datum = types.datum(
 17 :         [
 18 :           ["team", team]
 19 :         ],
 20 :         [
 21 :           ["batting average", float(bave)],
 22 :           ["Games", float(games)],
 23 :           ["bat", float(pa)],
 24 :           ["at-bats", float(atbat)],
 25 :           ["hits", float(hit)],
 26 :           ["home run", float(homerun)],
 27 :           ["RBI", float(runsbat)],
 28 :           ["stolen base", float(stolen)],
 29 :           ["walks", float(bob)],
 30 :           ["Hit by Pitch", float(hbp)],
 31 :           ["strike out", float(strikeout)],
 32 :           ["sacrifice", float(sacrifice)],
 33 :           ["double play", float(dp)],
 34 :           ["slugging percentage", float(slg)],
 35 :           ["on-base percentage", float(obp)],
 36 :           ["OPS", float(ops)],
 37 :           ["RC27", float(rc27)],
 38 :           ["XR27", float(xr27)]
 39 :         ]
 40 :         )
 41 :         # 3. Data training (update model)
 42 :         recommender.update_row(NAME, pname, datum)
 43 : 

**Analyze.py**

.. code-block:: python

 01 : #!/usr/bin/env python
 02 : # -*- coding: utf-8 -*-
 03 : 
 04 : import sys
 05 : from jubatus.recommender import client
 06 : from jubatus.recommender import types
 07 : 
 08 : NAME = "recommender_baseball";
 09 : 
 10 : if __name__ == '__main__':
 11 :     # 1. Connect to Jubatus Server
 12 :     recommender = client.recommender("127.0.0.1",9199)
 13 :     # 2. Prepare the data used for recommendation
 14 :     for line in open('dat/baseball.csv'):
 15 :       pname, team, bave, games, pa, atbat, hit, homerun, runsbat, stolen, bob, hbp, strikeout, sacrifice, dp, slg, obp, ops, rc27, xr27 = line[:-1].split(',')
 16 :       # 3. Recommendation by the model learnt
 17 :       sr = recommender.similar_row_from_id(NAME, pname , 4)
 18 :       # 4. Output result
 19 :       print "player ", pname,  " is similar to :", sr[1][0], sr[2][0], sr[3][0] 
 20 : 



--------------------------------
Explanation
--------------------------------

**npb_similar_player.json**

The configuration information is given by the JSON unit. Here is the meaning of each JSON filed.


* method

 Specify the algorithm used in classification. 
 This time, we specify it with "inverted_index", because we want to use an inverted index.
 Besides "inverted_index", we also support "minhash", "lsh" and "euclid_lsh".

* converter

 Specify the configurations in feature converter.
 In this example, we will set the "num_rules".
 
 "num_rules" are used to specify the extraction rules of numercial features.
 "key" is "*", it means all the "key" are taken into consideration, "type" is "num", it means the number(value) specified will be directly used as the input for training the model. 
 For example, if the "Batting average = 0.33", use 0.33 as the input; if the "RBI = 30", use 30 as the input.

 "string_rules" are used to specify the extraction rules of string features.
 Because string features are not used, we don't specify the "String_rules" here. 
   
* parameter

 Specify the parameters to be passed to the algorithm.
 The method specified here is "inverted_index", which doesn't need configuration.
 

**Update.py**

We explain the learning and recommendation processes in this example.

 To write the Client program for Recommender, we can use the RecommenderClient class defined in 'jubat.recommender'. There are two methods used in this program. The 'update_row' method for learning process, and the 'estimate' method for recommendation with the data learnt.
 
 1. Connect to Jubatus Server

  Connect to Jubatus Server (Row 12)
  Setting the IP addr., RPC port of Jubatus Server.


 2. Prepare the training data

  Prepare the Datum for model training at Jubatus Server.

  RecommenderClient puts the training data into a Datum List, and sends the data to update_row() methods for the model training.
  In this example, the training data is generated from the CSV file that privided by a baseball data website. 
  Baseball player information, including name, team, batting average, at-bats and hits.
  Figure below shows the training data.
  
 

  +-------------+--------------------------------------------------------+
  |ID(String)   |Datum                                                   |
  |             +--------------------------+-----------------------------+
  |             |TupleStringString(List)   |TupleStringDoubel(List)      |
  |             +------------+-------------+---------------+-------------+
  |             |key(String) |value(String)|key(String)    |value(double)|
  +=============+============+=============+===============+=============+
  |"Y. Oshima"  |"team"      |"Chunichi"   | | "Bat avg."  | | 0.31      |
  |             |            |             | | "Games"     | | 144       |
  |             |            |             | | "At-bat"    | | 631       |
  |             |            |             | | "At-bats"   | | 555       |
  |             |            |             | | "Hits"      | | 172       |
  |             |            |             | | "Home run"  | | 1         |
  |             |            |             | | "RBI"       | | 13        |
  |             |            |             | | "Steal"     | | 32        |
  |             |            |             | | "Walks"     | | 46        |
  |             |            |             | | "HBP"       | | 13        |
  |             |            |             | | "Strike out"| | 80        |
  |             |            |             | | "Sacrifice" | | 17        |
  |             |            |             | | "DP"        | | 7         |
  |             |            |             | | "SLG"       | | 0.368     |
  |             |            |             | | "OBP"       | | 0.376     |
  |             |            |             | | "OPS"       | | 0.744     |
  |             |            |             | | "RC27"      | | 5.13      |
  |             |            |             | | "XR27"      | | 4.91      |
  +-------------+------------+-------------+---------------+-------------+
  |"Y.Takahashi"|"team"      |"Giant"      | | "Bat avg."  | | 0.239     |
  |             |            |             | | "Games"     | | 130       |
  |             |            |             | | "At-bat"    | | 442       |
  |             |            |             | | "At-bats"   | | 368       |
  |             |            |             | | ･･･         | | ･･･       |
  |             |            |             | | ･･･         | | ･･･       |
  +-------------+------------+-------------+---------------+-------------+
  
  "Datum" is composed of key-value data which could be processed by Jubatus, and there are 2 types of key-value data format.
  In the first type, both the "key" and "value" are in string format (string_values); in the second one, the "key" is in string format, but the "value" is in numerical format (num_values).
  These two types are represented in TupleStringString class and TupleStringDouble class, respectively.
    
  | Please have a view of the first example data in this table. Because the "team" is in string format, it is stored in the first list of the TupleStringString class
  | in which, the key is set as "team", value is set as "Chunichi".   
  | Because other items are numerical, they are stored in the list of the TupleStringDouble class, in which
  | the first list's key is set as "Bat avg." and value is set as "0.31",
  | the second list's key is set as "Games" and value is set as "144",
  | the third list's key is set as "At-bat" and value is set as "631",
  | the fourth list's key is set as "At-bats" and value is set as "555".
  | ...
  | generate the final list by the last item "XR27".  
 
  The Datum of these Lists are generated for every players.
  Thus, the Datum, together with its player_id, are used as the training data.

  Here is the detailed process for making the training data in this sample.

  First, read the source file (CSV file) of the training data, and process the data line by line with the 'for' loop.(Row 14-42).
  Split the data of each line by the ',' mark (Row 15).
  Generate a datum of the items in different data type (Row 16-40).
  Now, the Datum for one player is created.

 3. Model Training (update learning model

  Input the training data generated in step.2 into the update_row() method (Row 42).
  The first parameter in update_row() is the unique name for task identification in Zookeeper.
  (use null charactor "" for the stand-alone mode)
  The second parameter specifies the unique ID for each players. In this example, the "name" of each player is used as the ID.
  The third parameter is the Datum for each player, that generated in Step 2.
  Now, the Datum of one player has been learnt. By looping the Steps 2 & 3 above, all the players' data in the CSV file will be learnt.


**Analyze.py**

 1. Connect to Jubatus Server

  Omitted here, because it is the same as Update.java.
  
 2. Prepare the data for recommendation

  The data used here is the unique play_ID in the previous training data, which is the players' names here.
  Player "name", the first item in the column, is input into the similar_row_from_id() method to get the recommended similar players.

 3. Recommendation by the model

  By inputting the player's name into the similar_row_from_id() method, the list of recommended players is return(Row 17).
  The first parameter in similar_row_from_id() is the unique name for task identification in Zookeeper.
  (use null charactor "" for the stand-alone mode)
  The second parameter specifies the unique ID for each players.
  The third parameter is the number of the most similar players to be returned. We specified "4" here to get the  most similar three players, because the top-1 is the player himself. 
  
 4.  Output the result

  The recommendation results are returned by the similar_row_from_id() method, and there are 4 players in the returned list. Because the first result is the input player himself, only the 2nd, 3nd and 4th results are output.
  Similar as Update.java, the Step 2.~4. are looped processed for each input players

------------------------------------
Run the sample program
------------------------------------

**[At Jubatus Server]**
 
 start "jubarecommender" process.


::

 $ jubarecommender --configpath npb_similar_player.json


**[At Jubatus Client]**

 Run the commands below.
 
::

 $ python update.py
 $ python analyze.py


**[Result]**


::

 player Nagano Hisayoshi is similar to : Yoshio Itoi, Milledge, Takumi Kuriyama
 player Yohei Oshima is similar to : Honda Yuichi, Ishikawa Hiroshi, Aranami Sho
 player Takashi Toritani is similar to : Saporo, Yoshio Itoi, Kazuhiro Wada
 player Hayato Sakamoto is similar to : Kakunaka Katsuya, Inaba Atsunori, Shogo Akiyama
 player Nakata Sho is similar to : Tadahito Iguchi, Arai Takahiro, Nakamura Norihiro
 …
 …(omitted)


