Java
==================

Here we explain the sample program of Recommender in Java. 

--------------------------------
Source_code
--------------------------------

In this sample program, we will explain 1) how to configure the learning-algorithms that used by Jubatus, with the example file 'npb_similar_player.json'; 2) how to train the model by 'Update.java' with the training data in 'baseball.csv' file, and how to recommend players with 'Analyze.java'. Here are the source codes of 'npb_similar_player.json', 'Update.java' and 'Analyze.java'.


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


**Update.java**

.. code-block:: java

 001 : package NpbSimilarPlayer;
 002 : 
 003 : import java.io.BufferedReader;
 004 : import java.io.File;
 005 : import java.io.FileNotFoundException;
 006 : import java.io.FileReader;
 007 : import java.io.IOException;
 008 : import java.util.ArrayList;
 009 : import java.util.Arrays;
 010 : import java.util.List;
 011 : 
 012 : 
 013 : import us.jubat.recommender.RecommenderClient;
 014 : import us.jubat.recommender.Datum;
 015 : import us.jubat.recommender.TupleStringDouble;
 016 : import us.jubat.recommender.TupleStringString;
 017 : 
 018 : public class Update {
 019 : 	public static final String HOST = "127.0.0.1";
 020 : 	public static final int PORT = 9199;
 021 : 	public static final String NAME = "NpbSimilarPlayer";
 022 : 	public static final String CSV_PATH = "./src/main/resources/baseball.csv";
 023 : 
 024 : 	// the column names in CSV
 025 : 	public static String[] CSV_COLUMN = {
 026 : 	        "Name", 
 027 :          "Team", 
 028 : 	        "batting average", 
 029 : 	        "Games",  
 030 : 	        "bat", 
 031 : 	        "at-bats", 
 032 : 	        "hits", 
 033 : 	        "home run", 
 034 : 	        "RBI", 
 035 : 	        "stolen base", 
 036 :	        "walks", 
 037 : 	        "Hit by Pitch", 
 038 : 	        "strike out", 
 039 : 	        "sacrifice", 
 040 : 	        "double play", 
 041 : 	        "slugging percentage", 
 042 : 	        "on-base percentage", 
 043 : 	        "OPS", 
 044 : 	        "RC27", 
 045 : 	        "XR27"  
 046 : 	        }; 
 047 : 
 048 : 	// column of String type
 049 : 	public static String[] STRING_COLUMN = {
 050 : 		"team"
 051 : 		};
 052 : 
 053 : 	// column of Double type
 054 : 	public static String[] DOUBLE_COLUMN = {
 055 : 		"Batting average",  
 056 : 		"Games",  
 057 : 		"bat",  
 058 : 		"at-bats",  
 059 : 		"hits",  
 060 : 		"home run",  
 061 : 		"RBI",  
 062 : 		"stolen base",  
 063 : 		"walks",  
 064 : 		"Hit by Pitch",  
 065 : 		"strike out",  
 066 : 		"sacrifice",  
 067 : 		"double play",  
 068 : 		"slugging percentage",  
 069 : 		"on-base percentage",  
 070 : 		"OPS",  
 071 : 		"RC27",  
 072 : 		"XR27"  
 073 : 		}; 
 074 : 
 075 : 	public void start() throws Exception {
 076 : 		// 1. Connect to Jubatus Server
 077 : 		RecommenderClient client = new RecommenderClient(HOST, PORT, 5);
 078 : 
 079 : 		// 2. Prepare the training data
 080 : 		Datum datum = null;
 081 : 
 082 : 		try {
 083 : 			File csv = new File(CSV_PATH); // CSV data file
 084 : 
 085 : 			BufferedReader br = new BufferedReader(new FileReader(csv));
 086 : 			List<String> strList = new ArrayList<String> ();
 087 : 			List<String> doubleList = new ArrayList<String> ();
 088 : 
 089 : 			String line = "";
 090 : 
 091 : 			// read data line by line, until the last one.
 092 : 			while ((line = br.readLine()) != null) {
 093 : 				strList.clear();
 094 : 				doubleList.clear();
 095 : 
 096 : 				// split the data in one line into items
 097 : 				String[] strAry = line.split(",");
 098 : 
 099 : 				for (int i=0; i<strAry.length; i++) {
 100 : 					if(Arrays.toString(STRING_COLUMN).contains(CSV_COLUMN[i])){
 101 : 						strList.add(strAry[i]);
 102 : 					} else if(Arrays.toString(DOUBLE_COLUMN).contains(CSV_COLUMN[i])){
 103 : 						doubleList.add(strAry[i]);
 104 : 					}
 105 : 				}
 106 : 				// make the training datum
 107 : 				datum = makeDatum(strList, doubleList);
 108 : 				// 3. Data training (update model)
 109 : 				client.update_row( NAME, strAry[0], datum);
 110 : 			}
 111 : 			br.close();
 112 : 
 113 : 		} catch (FileNotFoundException e) {
 114 : 			 // catch the exception in File object creation
 115 : 			 e.printStackTrace();
 116 : 		} catch (IOException e) {
 117 : 			 // catch the exception when closing BufferedReader object
 118 : 			 e.printStackTrace();
 119 : 		}
 120 : 		return;
 121 : 	}
 122 : 
 123 : 	// Create the lists with the name given in the Datum (for list)
 124 : 	private Datum makeDatum(List<String> strList, List<String> doublelist) {
 125 : 
 126 : 		Datum datum = new Datum();
 127 : 		datum.string_values = new ArrayList<TupleStringString>();
 128 : 		datum.num_values = new ArrayList<TupleStringDouble>();
 129 : 
 130 : 		for( int i = 0 ; i < strList.size() ; i++) {
 131 : 			TupleStringString data = new TupleStringString();
 132 : 			data.first = STRING_COLUMN[i];
 133 : 			data.second = strList.get(i);
 134 : 
 135 : 			datum.string_values.add(data);
 136 : 		}
 137 : 
 138 : 		try {
 139 : 			for( int i = 0 ; i < doublelist.size() ; i++) {
 140 : 				TupleStringDouble data = new TupleStringDouble();
 141 : 				data.first = DOUBLE_COLUMN[i];
 142 : 				data.second = Double.parseDouble(doublelist.get(i));
 143 : 
 144 : 				datum.num_values.add(data);
 145 : 			}
 146 : 		} catch (NumberFormatException e){
 147 : 			return null;
 148 : 		}
 149 : 
 150 : 		return datum;
 151 : 	}
 152 : 
 153 : 
 154 : 	// Main methods
 155 : 	public static void main(String[] args) throws Exception {
 156 : 		new Update().start();
 157 : 		System.exit(0);
 158 : 	}
 159 : }

**Analyze.java**

.. code-block:: java

 01 : package NpbSimilarPlayer;
 02 : 
 03 : import java.io.BufferedReader;
 04 : import java.io.File;
 05 : import java.io.FileNotFoundException;
 06 : import java.io.FileReader;
 07 : import java.io.IOException;
 08 : import java.util.ArrayList;
 09 : import java.util.List;
 10 : 
 11 : 
 12 : import us.jubat.recommender.RecommenderClient;
 13 : import us.jubat.recommender.TupleStringFloat;
 14 : 
 15 : public class Analyze {
 16 : 	public static final String HOST = "127.0.0.1";
 17 : 	public static final int PORT = 9199;
 18 : 	public static final String NAME = "NpbSimilarPlayer";
 19 : 	public static final String CSV_PATH = "./src/main/resources/baseball.csv";
 20 : 
 21 : 	public void start() throws Exception {
 22 : 		// 1. Connect to Jubatus Server
 23 : 		RecommenderClient client = new RecommenderClient(HOST, PORT, 5);
 24 : 
 25 : 		// 2. Prepare the data used for recommendation
 26 : 		 List<TupleStringFloat> rec = new  ArrayList<TupleStringFloat>();
 27 : 
 28 : 		 try {
 29 : 			File csv = new File(CSV_PATH); // CSV data file
 30 : 
 31 : 			BufferedReader br = new BufferedReader(new FileReader(csv));
 32 : 
 33 : 			// ead data line by line, until the last one.
 34 : 			String line = "";
 35 : 			while ((line = br.readLine()) != null) {
 36 : 
 37 : 				// split the data in one line into items
 38 : 				String[] strAry = line.split(",");
 39 : 
 40 : 				// 3. Recommendation by the model learnt
 41 : 				rec = client.similar_row_from_id(NAME, strAry[0], 4);
 42 : 
 43 : 				// 4. Output result
 44 : 				System.out.print("player " + strAry[0] + " is similar to : " + rec.get(1).first +
 45 : 						" " +  rec.get(2).first + " " + rec.get(3).first );
 46 : 				System.out.println();
 47 : 			}
 48 : 			br.close();
 49 : 
 50 : 		 } catch (FileNotFoundException e) {
 51 : 			 // capture the exception in File object creation.
 52 : 			 e.printStackTrace();
 53 : 		 } catch (IOException e) {
 54 : 			 // catch the exception when closing BufferedReader object捉
 55 : 			 e.printStackTrace();
 56 : 		 }
 57 : 
 58 : 		return;
 59 : 	}
 60 : 
 61 : 	// Main methods
 62 : 	public static void main(String[] args) throws Exception {
 63 : 		new Analyze().start();
 64 : 		System.exit(0);
 65 : 	}
 66 : }


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
 

**Update.java**

We explain the learning and recommendation processes in this example.

 To write the Client program for Recommender, we can use the RecommenderClient class defined in 'us.jubat.recommender'. There are two methods used in this program. The 'update_row' method for learning process, and the 'estimate' method for recommendation with the data learnt.
 
 1. Connect to Jubatus Server

  Connect to Jubatus Server (Row 33)
  Setting the IP addr., RPC port of Jubatus Server, and the connection waiting time.


 2. Prepare the training data

  Prepare the Datum for model training at Jubatus Server (Row 80).

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
  |                        ...
  | generate the final list by the last item "XR27".  
 
  The Datum of these Lists are generated for every players.
  Thus, the Datum, together with its player_id, are used as the training data.

  Here is the detailed process for making the training data in this sample.
  
  First, read the source file (CSV file) of the training data.
  Here, FileReader() and BuffererdReader() is used to read the items in CVS file line by line (Row 83-112).
  Split the data read from each line in CSV file, by the ',' mark (Row 76).
  Using the defined CSV item list (CSV_COLUMN),String item list (STRING_COLUMN) and Double item list (Double_COLUMN) to transfer the CSV data into strList or doubleList, if the item is in String or Double type (Row 99-105).
  
  The string item list and double item list in the arguments of [makeDatum] method are used to generate the TupleStringString list and TupleStringDouble list, respecitively (Row 124-151).
  At first, create the instance of Datum class component: "string_values" list and "num_values" list (Row 126-128).
  Next, generate the TupleStringString by reading the items from strList. The first element is the column name (as the key), and the second element is the value. The data is added into the string_values list (Row 130-136).
  The Double type items are processed in the similar way as String type items, to generate TupleStringDouble. Please note that the elements of num_values are added with type conversion, because the argument is of String type List while the num_values in Datum is of Double type (Row 142).
  Now, the Datum for one player is created.
  
 3. Model Training (update learning model

  Input the training data generated in step.2 into the update_row() method (Row 109).
  The first parameter in update_row() is the unique name for task identification in Zookeeper.
  (use null charactor "" for the stand-alone mode)
  The second parameter specifies the unique ID for each players. In this example, the "name" of each player is used as the ID.
  The third parameter is the Datum for each player, that generated in Step 2.
  Now, the Datum of one player has been learnt. By looping the Steps 2 & 3 above, all the players' data in the CSV file will be learnt.

**Analyze.java**

 1. Connect to Jubatus Server

  Omitted here, because it is the same as Update.java.
  
 2. Prepare the data for recommendation

  The data used here is the unique play_ID in the previous training data, which is the players' names here.
  Player "name", the first item in the column, is input into the similar_row_from_id() method to get the recommended similar players.

 3. Recommendation by the model

  By inputting the player's name into the similar_row_from_id() method, the list of recommended players is return(Row 41).
  The first parameter in similar_row_from_id() is the unique name for task identification in Zookeeper.
  (use null charactor "" for the stand-alone mode)
  The second parameter specifies the unique ID for each players
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

 Get the required package and Java client ready.
 
**[Result]**

::

 player Nagano Hisayoshi is similar to : Yoshio Itoi, Milledge, Takumi Kuriyama
 player Yohei Oshima is similar to : Honda Yuichi, Ishikawa Hiroshi, Aranami Sho
 player Takashi Toritani is similar to : Saporo, Yoshio Itoi, Kazuhiro Wada
 player Hayato Sakamoto is similar to : Kakunaka Katsuya, Inaba Atsunori, Shogo Akiyama
 player Nakata Sho is similar to : Tadahito Iguchi, Arai Takahiro, Nakamura Norihiro
 …
 …(omitted)
