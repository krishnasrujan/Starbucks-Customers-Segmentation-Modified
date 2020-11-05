# Starbucks Customer Segmentation
Segmenting customers based on their previous transactions using cluster analysis

![starbucks](https://user-images.githubusercontent.com/48923446/97382631-c8347c80-18f1-11eb-930e-7ad44deed0ff.jpg)

# Dataset
The data is contained in three files:

portfolio.json - containing offer ids and meta data about each offer (duration, type, etc.)

profile.json - demographic data for each customer

transcript.json - records for transactions, offers received, offers viewed, and offers completed

# Here is the schema and explanation of each variable in the files:

# portfolio.json
id (string) - offer id

offer_type (string) - type of offer ie BOGO(buy one get one), discount, informational

difficulty (int) - minimum required spend to complete an offer

reward (int) - reward given for completing an offer

duration (int) - time for offer to be open, in days

channels (list of strings)

This dataset contains information about all the offers (represented with offer id) and how they are sent to each customer(channels). What is the minimum cost required to spend to utilize the offer(difficulty) and what are the rewards for redeeming that offer(rewards). Offer type represents the type of offer, BOGO is buy one get one, discount and informational messages.

# profile.json
age (int) - age of the customer

became_member_on (int) - date when customer created an app account

gender (str) - gender of the customer (note some entries contain 'O' for other rather than M or F)

id (str) - customer id

income (float) - customer's income

This dataset contains customer information. Their age, income, gender, at what time they first visited starbucks and customer id(unique for each customer).

# transcript.json
event (str) - record description (ie transaction, offer received, offer viewed, etc.)

time (int) - time in hours since start of test. The data begins at time t=0

value - (dict of strings) - either an offer id or transaction amount depending on the record

This dataset contains information about the customers(already members) at what time they recieved the messages, what time they viewed the messages.The value column contains information about each customer whether they have utillized the offers properly. If they did, how much did they spend on that offer and if they did not utilize what is that offer id.

# Data Wrangling:

   # portfolio.json
      Used multilabel binarizer from sklearn to convert elements of list in channels column into individual columns.

   # profile.json
      Converted became_member_on into datetime and extracted year from it.

      Imputed income column with mean and gender column with mode.

      Dummied the categorical features.

   # transcript.json
      Each row in value column is a dictionary which contains either offer ids(for those who did not use the offer) or amount spent(for those who utilized the offer) as       values.
      
      Created two new datasets called offers and transactions. Those who utilized the offer come in transactions dataset , otherwise in offers dataset. Transaction has a new column called amount which is the amount spent by each customer on that offer. 
      
      Created 3 new columns for offers dataset which has information about the time at which customers received, viewed the offers and time at which the offer is completed.

   # Merging datasets
      Offers dataset is merged with profile and portfolio on person_id and offer_if respectively.
      
      Transactions dataset is merged with profile on person_id.
      
      These are the two datasets on which I will be working on.

# Offers Customer segmentation

There are  multiple events for each offer. Such as a customer receives two messages for same offer and customer views the offer multiple times. I calculated the total number of times that person received, viewed, completed the offer. 

Now since we have combined all the received, viewed, completed rows into single row, we can now remove duplicate offers for customers by taking the maximum value of each column.

Many people might have viewed the offer after the offer is completed. To check that I have created a column called viewed_on_time which tells if a person viewed the offer on time or not.

Dropped misattributions from the dataset. That is dropping rows which have viewed time> received time, completed time > viewed time and customer did not view the offer and the offer is completed.

Our only target are the persons who viewed the offer on time but did not buy anything. so our final dataset consists information about all the people who have received offers on time, viewed on time but did not take any action.

Dummied the offer type column and dropped. Dropping time, offer received, offer completed, offer viewed as these are compensated in received time, viewed time,         completed time.

Applied principal component analysis on ths dataset and retained 88 percent variance with 9 principle components.

<img width="270" alt="pcomponents" src="https://user-images.githubusercontent.com/48923446/98115223-7cbf3700-1ecc-11eb-9724-f5ba2cf0160b.PNG">

Now I will be doing cluster analysis on this data to understand the different types of customers.

   # Cluster Analysis
   Now we have to figure out right number of clusters for doing clustering. This is done by k elbow method.
   Manually doing k-elbow:
   
   <img width="383" alt="manual kelbow" src="https://user-images.githubusercontent.com/48923446/97405009-07c48e00-191d-11eb-9acb-9d55c7631b52.PNG">
   
   By seeing the plot we can decide that the right number of clusters are 3. Anyways we will also cross check it with a python function.
   
   K elbow using KElbowVisualizer from yellowbrick:
   
   <img width="391" alt="lib kelbow" src="https://user-images.githubusercontent.com/48923446/97405308-7275c980-191d-11eb-80a8-15fbdc3b79a4.PNG">

   K means clustering with n_clusters=3:
   
   <img width="382" alt="kmeans" src="https://user-images.githubusercontent.com/48923446/98114498-58af2600-1ecb-11eb-9ed7-fa708965186e.PNG">
   
   DBSCAN with eps=3.5 and n_samples=150:
   
   <img width="375" alt="dbscan" src="https://user-images.githubusercontent.com/48923446/98114510-5cdb4380-1ecb-11eb-860c-7e15bd88b21d.PNG">
   
   Cluster analysis:
   
   <img width="444" alt="segmenting" src="https://user-images.githubusercontent.com/48923446/98114521-5f3d9d80-1ecb-11eb-99ae-6a20f94b4f2e.PNG">
   
   Segment 1
   
      This segment includes males and females whose mean income is very high. They mostly received informational messages and received very less bogo and discout       offers. Their conversion rates is also less compared to other segments. If they are sent some BOGO offers and discount offers with more rewards they might utilize the offers.

   Segment 2   
   
      All customers in this segment have registered as other gender. Their mean income is slightly less compared to 1st segment. They mostly receive BOGO and receive very less discount and informational offers. Their conversion rates is less and rewards are more. It's better not to focus marketing on this segment.

   Segment 3
   
      This segment includes males and females whose mean income is very high. They only receive discount offers with high conversion rates. If they are sent some discount offers with low conversion rates they might convert the offers.

Number of customers per cluster :

<img width="331" alt="cluster population" src="https://user-images.githubusercontent.com/48923446/98114528-619ff780-1ecb-11eb-8f22-0489c0e6565b.PNG">
