
**Twitter Sentiment Analysis with GridDB - Part 1**




![alt_text](images/image1.png "image_tooltip")


**Introduction**

We often need to calculate the sentiment of enormous text data which are generated daily to track the user’s opinion on a particular topic. We can visualize the sentiment values for a particular geographic location which helps businesses for better decision making.
In this article, we will load the tweet dataset into GridDB, perform sentiment analysis on the dataset, and visualize using matplotlib. 

**Prerequisites** 

We will use the GridDb instance with the python3, matplotlib, pandas for sentiment calculation and Visualization. The tweet dataset is obtained from web scraping of tweets from 2013-2017 on the topic of “Rana Plaza”. 

**Dataset Schema**

In 2013, the Rana Plaza building collapse in Bangladesh killed more than 1100 fashion workers and tarnished the image of big fashion brands like Zara, H&M, Gap, Benetton, etc. To study how the consumers reacted to these fashion brands, we will perform sentiment analysis on the tweets mined from 2013-2018, on the topic of “Rana Plaza”.
We will consider the below attributes from the excel dataset file to be used in the GridDB container. 
 
| Field Name  | Data Type(GridDB)  |Notes|
|---|---|---|
|Serial No   |INTEGER   |   |
|Screen Name   |STRING   |Twitter Author Name   |
|Twitter ID   |STRING   |Twitter handle   |
|Tweet   |STRING   |Tweet text   |
|Date   |STRING   |   |


**Extracting Data:**

We will read the excel(.xlsx) file and iterate through each row and store it using pandas, in a data frame. Please note that here the dataset excel file contains more than one sheets so we need to iterate through each sheet. Below is the code snippet to read the sample dataset using pandas.


```
import pandas as pd
import xlrd

list_sheetnames = pd.ExcelFile("sample_dataset.xls").sheet_names
total_num_sheets = len(list_sheetnames)

for i in range(total_num_sheets):
   sheet_read = pd.read_excel("sample_dataset.xls", list_sheetnames[i])
   tweet_data_df = pd.DataFrame(sheet_read)
   print(tweet_data_df[:5])
```


Further, let us fetch all the columns of sample excel file, and save it in form of a list.



```
tweets_list = sheet_read["Tweet"].values.tolist()
tweet_date_list = sheet_read["Date"].values.tolist()
tweet_author_names_list = sheet_read["Screen Name"].values.tolist()
tweet_author_handle_list = sheet_read["Twitter Id"].values.tolist()
print(tweet_author_handle_list[:5])
```


The output of the above code snippet will look like this. 



```
/Users/vj/PycharmProjects/sentiment_analysis_blogs/venv/bin/python /Users/vj/PycharmProjects/sentiment_analysis_blogs/main.py
['VicUnions', 'ituc', 'shopjanery', 'EthicalBrandMkt', 'CattyDerry']
['Vic Trades Hall\u200f\xa0', 'ITUC\u200f\xa0', 'Jane Pearson\u200f\xa0', 'Ethical Brand Marketing\u200f\xa0', 'CATTY DERRY\u200f\xa0']

Process finished with exit code 0
```


In the next step, we will clean and preprocess the data so that sentiment operation can be performed effectively on them. The tweet text must be cleaned by removing links and special characters so that the sentiment analysis can be effectively performed upon them.



```
# a function to clean the tweets using regular expression
def clean_tweet(tweet):
   '''
   Utility function to clean the text in a tweet by removing
   links and special characters using regex.
   '''
   return ' '.join(re.sub("(@[A-Za-z0-9]+)|([^0-9A-Za-z \t])|(\w+:\/\/\S+)", " ", tweet).split())
```



**Inserting Data into GridDB container**

We will feed the data stored in the data frame to a GridDB container using put or multi-put methods. We will create GridDB containers to store data, each container will contain the data of the corresponding sheet-name of the sample dataset. 
```
factory = griddb.StoreFactory.get_instance()

# Get GridStore object
# Provide the necessary arguments
gridstore = factory.get_store(
   host=argv[1],
   port=int(argv[2]),
   cluster_name=argv[3],
   username=argv[4],
   password=argv[5]
)

curr_sheet_griddb_container = curr_sheet

# create collection for the tweet data in the sheet
tweet_data_container_info = griddb.ContainerInfo(circuits_container,
                                                [["sno", griddb.Type.INTEGER],
                                                 ["twitter_name", griddb.Type.STRING],
                                                 ["twitter_id", griddb.Type.STRING],
                                                 ["tweet", griddb.Type.STRING],
                                                 ["date", griddb.Type.STRING]],
                                                griddb.ContainerType.COLLECTION, True)

tweets_columns = gridstore.put_container(tweet_data_container_info)
```

In the next step, we will insert the dataframe in the row using put_rows function.


```
# Put rows
# Pass the data frames as param
tweets_columns.put_rows(tweet_data_df)
```

We have successfully inserted the data in griddb collections in the given dataset schema, after preprocessing it. 

**Retrieving Data from the Containers**

We will fetch the data using get_container method and then extract the data by querying that collection


```
# Define the container names
tweet_dataaset_container = excel_sheet_name

# Get the containers
tweet_data = gridstore.get_container(tweet_dataaset_container)

# Fetch all rows - tweet_container
query = tweet_data.query("select *")
rs = query.fetch(False)
print(f"{tweet_dataaset_container} Data")

# Iterate and create a list
retrieved_data = []
while rs.has_next():
   data = rs.next()
   retrieved_data.append(data)

print(retrieved_data)

# Convert the list to a pandas data frame
tweet_dataframe = pd.DataFrame(retrieved_data,
                                 columns=['sno', 'twitter_name', 'twitter_id', 'tweet', 'date'])

# Get the data frame details
print(tweet_dataframe)
tweet_dataframe.info()
```


The used query will fetch all those tweets data from 2013- 2018 which is in dataset. This is a basic TQL statement to select the entire rows from the tweet data container.



```
query = tweet_data.query("select *")
```


We can pass any suitable query to fetch data for some conditions. We can further map the list to a data-frame.

**Conclusion:**

In this part 1 of the blog, we discussed how to preprocess the tweets data for sentiment analytics, from the excel file. Further, we used GridDB containers to create collections and queried throughout to extract the required attributes.

**Source Code**

[GitHub](https://github.com/6vedant/SentimentAnalysisPart1)
