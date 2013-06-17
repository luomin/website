Recommender
===================

In this sample program, we will introduce how to use the Recommender function 'jubarecommender' through the Jubatus Client.

By using Recommender, we can recommend the similar data or data with the similar attribute. This function is useful for the EC site products recommendation or the linked ads recommendation in search site.

-----------------------------------
Abstract of sample program
-----------------------------------

In this sample, we will use a [npb_similar_player] program to study the performance records of 2012 japanese baseball league players, and recommend the players with similar performance.

At first, please download the `Baseball league player data <http://baseball-data.com/>`_, from (`baseball.csv <https://raw.github.com/jubatus/jubatus-example/master/npb_similar_player/dat/baseball.csv>`_). It is used to training the recommend model at the server site. 

Next, input other players' data to the Jubatus server for their similar players data as the recommendation results. The input data are read from an external file and processed in the same way as the .csv file above. 

For example, if the player [Sho Nakata]'s data is input for its recommendation, the result is returned as: [player Sho Nakata is similar to : Tadahito Iguchi; Takahiro Arai; Norihiro Nakamura], the three infielders who have the most similar performance as Sho Nakata. 

--------------------------------
Processing flow 
--------------------------------

Main flow of using Jubatus Client

* Upadate

 1. Connection settings to Jubatus Server

  Setting the HOST, RPC port of Jubatus Server

 2. Prepare the training data

  Get all the filed players' data from the baseball.csv file.

 3. Data training (update the model)

  Use the update_row() method to read the players' data line by line, and send to the server site to train the model there.

* Analyze

 1. Connection settings to Jubatus Server

  Setting the HOST, RPC port of Jubatus Server

 2. Prepare the data for recommendation

  Generate the players' data to be analyzed.

 3. Recommendation based on the model at Jubatus server 

  Get the recommendation results of the players in Step 2, by using similar_row_from_id().

 4. Output the result

  Output the results returned from similar_row_from_id().

--------------------------------
Sample program
--------------------------------

.. toctree::
   :maxdepth: 2

   recommender_python
   recommender_ruby
   recommender_java
