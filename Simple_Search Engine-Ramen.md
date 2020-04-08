# INFO 624 Final Project </br> Search Engine Building - Instant Ramen Noodles Rating 
Siyi Lin


# Why: Explain the purpose of the search engine and what difference you can make.

Our project is to build a simple search engine for the ramen enthusiasm. Reman: as one of the best foods in this world is becoming more and more popular in worldwide range. Not only for Japanese people but also for people with different races and nations. In this growing trend, reman industry is getting popular; however, it’s becoming hard for reman fans to find the reman product they want. We want to build a simple search engine to help customer make their decisions easier based on the previous reviews.

# What: Describe the data and domain in which you build the search engine index.

The dataset is generated from Ramen Rater, a product review website for the hardcore ramen eater, with over 2500 reviews up-to-date. The dataset is avaliable on Kaggle. [The linke: Ramen Ratings](https://www.kaggle.com/residentmario/ramen-ratings) 

It contains 7 attributes with 2580 instances. Each instance records a single instant ramen product represented by the following aspects:
* Review Numbers (The more recent reviews,  the higher number is) 
* Brand (Vendor/Manufacturer)
* Variety (Product name)
* Country
* Style (Cup? Bowl? Tray?) 
* Stars (on a 5-point scale)


# How do we conduct the project
There are two team members, Shuai Teng and Siyi Lin, in this project. Each member will create 3 use cases and explore the dataset by Kibana, later we will see our results.

Three use cases for Siyi Lin & the analysis with results

1. find any noodle with oyster sauce or in oyster flavor
2. find any Japeness curry noodle  
3. find any Japeness curry noodle in the cup



Three use cases for Teng Shuai & the analysis with results

1. Online food order  
2. Reman noodles restaurant 
3. Homemade reman

Due to the different working environment for two of us, our report is merged from two format into one. 

The following is from Siyi Lin:

# Who: Explain whom your search engines may serve and their basic information needs.
This search engine is open for everyone acccess.

1. USE CASE #1 - The eater is looking for any noodle with oyster sauce or in oyster flavor
2. USE CASE #2 - The eater is looking for any Japeness curry noodle
3. USE CASE #3 - The eater is looking for any Japeness curry noodle in the cup, so they can enjoy the reman without cooking it at home 



# How - Decisions and steps to build the engine

## 1 Describe the data fields related to the project and how they should be analyzed and indexed. Select proper data types, analyzers, filters, etc. for each field, and provide your rationale.

After reviewing our dataset, we believe the related data fields to the project are:

* Brand
* Variety
* Style
* Country
* Stars


The following table is about all the information about the data types, analyzers, filters

| name    | datatype      | analyzer   |
|:-------:|:-------------:|:----------:|
|Brand    | text          | english    |
|Variety  | text          | synonym_analyzer* |
|Style    | text          | standard   |
|Country  | keyword       | standard   |
|Stars    | scaled_float* | N/A        |


* scaled_float

Taking floating-point data into integer will increase the search efficiency.  The *scaled_float* with the *scaling_factor* of 100 means that each two decimal number will be read as an integer by Elasticsearch. For instance, 3.75 will interpret as 375.


```json
      "Stars": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
```




[Reference: scaled_float](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)

* synonym_analyzer: 

We noticed that the word *flavor* and the word *flovour* are used interchangeablely. As so, we paired these two words togehter and set as a filter in the *synonym_analyzer* 

```json
{
  "index" : {
      "analysis" : {
          "filter" : {
              "synonym_filter" : {
                  "type" : "synonym",
                  "synonyms" : ["flavour, flavor"]
              }
          },
          "analyzer" : {
              "synonym_analyzer" : {
                  "tokenizer" : "standard",
                  "filter" : ["lowercase", "synonym_filter"] 
              }
          }
      }
  }
}
```
[Referencec: Synonym token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html)



## 2. Describe your search queries in terms the use cases (needs):

### What keywords and query structure should be used? <br> What fields should be searched for potential matches? <br> What fields should be included in the scoring (ranking)? In what manner?

|UC# | Information Needs | Keywords | Fields for Match| Field for Ranking
|----|-------------------|----------|--------|---------|
|1   | looking for any noodle with oyster sauce or in oyster flavor| oyster sauce flavor | Variety | Variety | 
|2   | looking for any Japeness curry noodle| japeness curry | Variety | Variety, Country |
|3   | looking for any cup Japeness curry noodle, so they can enjoy the reman without cooking it at home | curry, Japan, Cup | Variety, Country, Style | Variety, Country, Style |




## 3. Select related similarity, scoring, and boosting methods accordingly


| name    | Similarity   | 
|:-------:|:------------:|
|Brand    | boolean      |
|Variety  | my_bm25*     | 
|Style    | boolean      |
|Country  | boolean      |
|Stars    | N/A          | 


* my_bm25

The name of the noodle is usually short and straightforward; no word will repeat twice. Due to the fact just mentioned, TF/IDF doesn't fit for our project. We want to use BM25 to check the relavence for scoring. 

Setting b to 0 in people2: the length — or total number of terms in the document — doesn’t affect the scoring; only the count and relevance of the matching terms. 

For k1, we prefer set it on the lower side since the name of reman noodle contains only a few words and it's unlikely to put a word without being highly related to the noodle as a subject.

```json
{
  "settings": {
    "index":{
      "similarity":{
        "my_bm25":{
          "type": "BM25",
          "k1": 0.01,
          "b": 0
        }
      }
    }
  }
}
```

[Reference1: Improved Text Scoring with BM25](https://www.elastic.co/elasticon/conf/2016/sf/improved-text-scoring-with-bm25) 

[Reference2:The BM25 Algorithm and its Variables](https://www.elastic.co/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables)




## 4. Create your index, mappings, and load data.
Since our data is in the .csv format, the steps of data loading and mapping as following:

1. create the setting of the index
2. upload .cvs file into **XLSX Import**
3. create the mapping for the index

Seting Index
```json
PUT  /sl3633_info624_201902_project2
{
  "settings": {
    "index":{
      "similarity":{
        "my_bm25":{
          "type": "BM25",
          "k1": 0.01,
          "b": 0
        },
        "my_dfr":{
          "type": "DFR",
          "basic_model": "g",
          "after_effect": "l",
          "normalization": "h2",
          "normalization.h2.c": "3.0"
        }
      },
      "analysis" : {
        "filter" : {
            "synonym_filter" : {
                "type" : "synonym",
                "synonyms" : [
                  "flavour, flavor"
                ]
            }
        },
        "analyzer" : {
            "synonym_analyzer" : {
                "tokenizer" : "standard",
                "filter" : [
                  "lowercase", "synonym_filter"
                ] 
            }
        }
      }
    }
  }
}
```


Mapping the index:
```json
PUT /sl3633_info624_201902_project2/_mapping
{
  "properties" : {
    "Brand" : {
      "type" : "text",
      "analyzer": "english",
      "similarity": "boolean"
    }
  }
}

PUT /sl3633_info624_201902_project2/_mapping
{
  "properties" : {
    "Country" : {
      "type" : "keyword",
      "analyzer": "standard",
      "similarity": "boolean"
    }
  }
}

PUT /sl3633_info624_201902_project2/_mapping
{
  "properties" : {
    "Stars" : {
      "type": "scaled_float",
      "scaling_factor": 100
    }
  }
}

PUT /sl3633_info624_201902_project2/_mapping
{
  "properties" : {
    "Style" : {
      "type" : "text",
      "analyzer": "standard",
      "similarity": "boolean"
    }
  }
}

PUT /sl3633_info624_201902_project2/_mapping
{
  "properties" : {
    "Variety" : {
      "type" : "text",
      "analyzer" : "synonym_analyzer",
      "similarity": "my_bm25"
    }
  }
}
```



# How Good - Test and evaluate your search engine index

## 1 Test the query for each use case (information need); limit the number of hits to the top 10

* UC#1: looking for any noodle with oyster sauce or in oyster flavor <br>
Keywords: "oyster sauce flavor"<br>
Field: *Variety* field 

```json
GET /sl3633_info624_201902_project2/_search
{
  "from":0, "size":10,
 "query":{
   "multi_match": {
    "query": "oyster sauce flavor", 
    "fields": ["Variety"] 
   }
 }
}
```
The results:
```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 846,
      "relation" : "eq"
    },
    "max_score" : 10.532438,
    "hits" : [
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "426",
        "_score" : 10.532438,
        "_source" : {
          "Brand" : "A-Sha Dry Noodle",
          "Variety" : "Chow Mein Oyster Sauce BBQ Flavor",
          "Style" : "Tray",
          "Country" : "Taiwan",
          "Stars" : "2.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "596",
        "_score" : 9.262579,
        "_source" : {
          "Brand" : "A-Sha Dry Noodle",
          "Variety" : "Quinoa Noodle With Oyster Sauce And Vegetables",
          "Style" : "Pack",
          "Country" : "Taiwan",
          "Stars" : "4.25"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "2084",
        "_score" : 7.621714,
        "_source" : {
          "Brand" : "Hsin Tung Yang",
          "Variety" : "Tiny Noodle With Oyster Flavor",
          "Style" : "Pack",
          "Country" : "Taiwan",
          "Stars" : "0.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "564",
        "_score" : 6.351855,
        "_source" : {
          "Brand" : "Pulmuone",
          "Variety" : "Noodle With Spicy Oyster Soup",
          "Style" : "Pack",
          "Country" : "South Korea",
          "Stars" : "4.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "232",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Cup Noodles BIG XO Sauce Seafood Flavour",
          "Style" : "Cup",
          "Country" : "Hong Kong",
          "Stars" : "4.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "261",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Sau Tao",
          "Variety" : "Black Pepper XO Sauce Flavour",
          "Style" : "Bowl",
          "Country" : "Hong Kong",
          "Stars" : "5.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "282",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Demae Ramen Spicy Xo Sauce Seafood Flavour Instant Noodle",
          "Style" : "Pack",
          "Country" : "Hong Kong",
          "Stars" : "4.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "289",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Nupasta Bacon In Carbonara Sauce Flavour Instant Noodle",
          "Style" : "Pack",
          "Country" : "Hong Kong",
          "Stars" : "5.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "305",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Goku-Uma",
          "Variety" : "Ramen Noodles Soy Sauce Flavor",
          "Style" : "Bowl",
          "Country" : "USA",
          "Stars" : "3.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "332",
        "_score" : 4.180584,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Demae Iccho Spicy XO Sauce Seafood Flavour",
          "Style" : "Pack",
          "Country" : "Hong Kong",
          "Stars" : "4.00"
        }
      }
    ]
  }
}
```


* UC#2: looking for any Japeness curry noodle <br>
Keywords: "japeness curry"<br>
Field: *Variety* field


```json

GET /sl3633_info624_201902_project2/_search
{
  "from":0, "size":10,
 "query":{
   "multi_match": {
    "query": "japanese curry", 
    "fields": ["Variety"] 
   }
 }
}
```



The result:
```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 158,
      "relation" : "eq"
    },
    "max_score" : 7.2289004,
    "hits" : [
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "60",
        "_score" : 7.2289004,
        "_source" : {
          "Brand" : "Takamori",
          "Variety" : "Hearty Japanese Style Curry Udon",
          "Style" : "Pack",
          "Country" : "Japan",
          "Stars" : "5.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1365",
        "_score" : 7.2289004,
        "_source" : {
          "Brand" : "Doll",
          "Variety" : "Hello Kitty Dim Sum Noodle Japanese Curry Flavour",
          "Style" : "Cup",
          "Country" : "Hong Kong",
          "Stars" : "3.50"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1389",
        "_score" : 7.2289004,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Soba Curry Noodles With Japanese Yakisoba Sauce",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1414",
        "_score" : 7.2289004,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Donbei Curry Udon (West Japanese)",
          "Style" : "Bowl",
          "Country" : "Japan",
          "Stars" : "3.25"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1887",
        "_score" : 7.2289004,
        "_source" : {
          "Brand" : "JFC",
          "Variety" : "Japanese Style Noodle Curry",
          "Style" : "Bowl",
          "Country" : "Japan",
          "Stars" : "4.50"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "112",
        "_score" : 4.205274,
        "_source" : {
          "Brand" : "Myojo",
          "Variety" : "Udon Japanese Style Noodles With Soup Base Hot & Sour Flavor",
          "Style" : "Pack",
          "Country" : "USA",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "151",
        "_score" : 4.205274,
        "_source" : {
          "Brand" : "Dream Kitchen",
          "Variety" : "Udon Japanese Style Fresh Noodle",
          "Style" : "Bowl",
          "Country" : "USA",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "179",
        "_score" : 4.205274,
        "_source" : {
          "Brand" : "Goku-Uma",
          "Variety" : "Yakisoba Japanese Style Noodle",
          "Style" : "Bowl",
          "Country" : "USA",
          "Stars" : "4.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "315",
        "_score" : 4.205274,
        "_source" : {
          "Brand" : "Shirakiku",
          "Variety" : "Karami Ramen Spicy Chili Flavour Japanese Style Noodle With Soup Base",
          "Style" : "Pack",
          "Country" : "USA",
          "Stars" : "2.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "374",
        "_score" : 4.205274,
        "_source" : {
          "Brand" : "Roland",
          "Variety" : "Ramen Japanese Style Quick-Cooking Alimentary Paste With Chicken Artificially Flavored Soup Base",
          "Style" : "Pack",
          "Country" : "USA",
          "Stars" : "0.00"
        }
      }
    ]
  }
}
```




* UC#3: looking for any Japeness curry noodle <br>
Keywords: "japeness curry", "cup" 
Field:  *Variety* and *Styles* field

```json
GET /sl3633_info624_201902_project2/_search
{
  "from":0, "size":10,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "Variety": "japaness curry"
          }
        },
        {
          "match": {
            "Style": "Cup"
          }
        }
      ]
    }
  }
}

```

The result:
```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 35,
      "relation" : "eq"
    },
    "max_score" : 8.228901,
    "hits" : [
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1365",
        "_score" : 8.228901,
        "_source" : {
          "Brand" : "Doll",
          "Variety" : "Hello Kitty Dim Sum Noodle Japanese Curry Flavour",
          "Style" : "Cup",
          "Country" : "Hong Kong",
          "Stars" : "3.50"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1389",
        "_score" : 8.228901,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Soba Curry Noodles With Japanese Yakisoba Sauce",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "538",
        "_score" : 5.205274,
        "_source" : {
          "Brand" : "Doll",
          "Variety" : "Hello Kitty Dim Sum Noodle Japanese Soy Sauce Flavour",
          "Style" : "Cup",
          "Country" : "Hong Kong",
          "Stars" : "3.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1092",
        "_score" : 5.205274,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Soba Chili Noodles With Japanese Yakisoba Sauce",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.50"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1214",
        "_score" : 5.205274,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Soba Teriyaki Noodles With Japanese Yakisoba Sauce",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "1360",
        "_score" : 5.205274,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Soba Classic Noodles With Japanese Yakisoba Sauce",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "39",
        "_score" : 4.0236263,
        "_source" : {
          "Brand" : "KOKA",
          "Variety" : "Curry Flavour Instant Noodles",
          "Style" : "Cup",
          "Country" : "Singapore",
          "Stars" : "5.00"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "89",
        "_score" : 4.0236263,
        "_source" : {
          "Brand" : "Nissin",
          "Variety" : "Cup Noodles Curry",
          "Style" : "Cup",
          "Country" : "Germany",
          "Stars" : "3.75"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "113",
        "_score" : 4.0236263,
        "_source" : {
          "Brand" : "KOKA",
          "Variety" : "Signature Curry Flavor Instant Noodles",
          "Style" : "Cup",
          "Country" : "Singapore",
          "Stars" : "3.50"
        }
      },
      {
        "_index" : "sl3633_info624_201902_project2",
        "_type" : "_doc",
        "_id" : "136",
        "_score" : 4.0236263,
        "_source" : {
          "Brand" : "KOKA",
          "Variety" : "Creamy Soup Witrh Crushed Noodles Curry Flavor",
          "Style" : "Cup",
          "Country" : "Singapore",
          "Stars" : "5.00"
        }
      }
    ]
  }
}
```



## 2 Examine the results and judge each document’s relevance in terms of the information need
For all three cases, we take the first 10 results. Then based on our needs, we check the relevance of all 10 documents we retrieved manually. Here are the results. 1 means relevant; 0 means non-relevant.

* UC1 Relevant Check Table:

|      |                                                           |           |           | 
|------|-----------------------------------------------------------|-----------|-----------| 
| Id   | Variety                                                   | Score     | Relevant? | 
| 426  | Chow Mein Oyster Sauce BBQ Flavor                         | 10.532438 | 1         | 
| 596  | Quinoa Noodle With Oyster Sauce And Vegetables            | 9.262579  | 1         | 
| 2084 | Tiny Noodle With Oyster Flavor                            | 7.621714  | 1         | 
| 564  | Noodle With Spicy Oyster Soup                             | 6.351855  | 1         | 
| 232  | Cup Noodles BIG XO Sauce Seafood Flavour                  | 4.180584  | 0         | 
| 261  | Black Pepper XO Sauce Flavour                             | 4.180584  | 0         | 
| 282  | Demae Ramen Spicy Xo Sauce Seafood Flavour Instant Noodle | 4.180584  | 0         | 
| 289  | Nupasta Bacon In Carbonara Sauce Flavour Instant Noodle   | 4.180584  | 0         | 
| 305  | Ramen Noodles Soy Sauce Flavor                            | 4.180584  | 0         | 
| 332  | Demae Iccho Spicy XO Sauce Seafood Flavour                | 4.180584  | 0         |    

4 documents are relevant for case 1; 846 documents are retrieved.

* UC2 Relevant Check Table:

|      |                                                                                                  |           |           | 
|------|--------------------------------------------------------------------------------------------------|-----------|-----------| 
| Id   | Variety                                                                                          | Score     | Relevant? | 
| 60   | Hearty Japanese Style Curry Udon                                                                 | 7.2289004 | 1         | 
| 1365 | Hello Kitty Dim Sum Noodle Japanese Curry Flavour                                                | 7.2289004 | 1         | 
| 1389 | Soba Curry Noodles With Japanese Yakisoba Sauce                                                  | 7.2289004 | 1         | 
| 1414 | Donbei Curry Udon (West Japanese)                                                                | 7.2289004 | 1         | 
| 1887 | Japanese Style Noodle Curry                                                                      | 7.2289004 | 1         | 
| 112  | Udon Japanese Style Noodles With Soup Base Hot & Sour Flavor                                     | 4.205274  | 0         | 
| 151  | Udon Japanese Style Fresh Noodle                                                                 | 4.205274  | 0         | 
| 179  | Yakisoba Japanese Style Noodle                                                                   | 4.205274  | 0         | 
| 315  | Karami Ramen Spicy Chili Flavour Japanese Style Noodle With Soup Base                            | 4.205274  | 0         | 
| 374  | Ramen Japanese Style Quick-Cooking Alimentary Paste With Chicken Artificially Flavored Soup Base | 4.205274  | 0         |    

5 documents are relevant for case 2; 158 documents are retrieved.


* UC3 Relevant Check Table:

|      |                                                       |       |           |           | 
|------|-------------------------------------------------------|-------|-----------|-----------| 
| Id   | Score                                                 | Style | Variety   | Relevant? | 
| 1365 | Hello Kitty Dim Sum Noodle Japanese Curry Flavour     | Cup   | 8.228901  | 1         | 
| 1389 | Soba Curry Noodles With Japanese Yakisoba Sauce       | Cup   | 8.228901  | 1         | 
| 538  | Hello Kitty Dim Sum Noodle Japanese Soy Sauce Flavour | Cup   | 5.205274  | 0         | 
| 1092 | Soba Chili Noodles With Japanese Yakisoba Sauce       | Cup   | 5.205274  | 0         | 
| 1214 | Soba Teriyaki Noodles With Japanese Yakisoba Sauce    | Cup   | 5.205274  | 0         | 
| 1360 | Soba Classic Noodles With Japanese Yakisoba Sauce     | Cup   | 5.205274  | 0         | 
| 39   | Curry Flavour Instant Noodles                         | Cup   | 4.0236263 | 0         | 
| 89   | Cup Noodles Curry                                     | Cup   | 4.0236263 | 0         | 
| 113  | Signature Curry Flavor Instant Noodles                | Cup   | 4.0236263 | 0         | 
| 136  | Creamy Soup Witrh Crushed Noodles Curry Flavor        | Cup   | 4.0236263 | 0         | 



2 documents are relevant for case 3; 35 documents are retrieved.

<br>

## 3 Provide a formal evaluation in terms of metrics such as precision, recall, F1, and/or nDCG

UC1: 4 documents are relevant; 846 documents are retrieved.

UC2: 5 documents are relevant; 158 documents are retrieved.

UC3: 2 documents are relevant; 35 documents are retrieved.

* Precision and Recall

Since we have 2580 documents in this dataset, we aren't able to check all relavant files manually by ourselves. We don't know the total amount of the relevant or non-relevant documents. 


|               |          |              | 
|---------------|----------|--------------| 
| UC 1          | Relevant | Non-relevant | 
| Retrieved     | 4 (tp)   | 842(fp)      | 
| Not Retrieved | NA (fn)  | NA (tn)      | 

Precision P = tp/(tp + fp) = 4/(4+842) = 0.0047  <br>
Recall R = tp/(tp + fn); we need to know more information in relevant for the rest of documents <br>

&nbsp;

|               |          |              | 
|---------------|----------|--------------| 
| UC 2          | Relevant | Non-relevant | 
| Retrieved     | 5 (tp)   | 153(fp)      | 
| Not Retrieved | NA (fn)  | NA (tn)      | 

Precision P = tp/(tp + fp) = 5/(5+153) = 0.0316 <br>
Recall R = tp/(tp + fn) we need to know more information in relevant for the rest of documents <br>

&nbsp;

|               |          |              | 
|---------------|----------|--------------| 
| UC 3          | Relevant | Non-relevant | 
| Retrieved     | 2 (tp)   | 33 (fp)      | 
| Not Retrieved | NA (fn)  | NA (tn)      | 

Precision P = tp/(tp + fp) = 2/(2+33) = 0.0571 <br>
Recall R = tp/(tp + fn)  we need to know more information in relevant for the rest of documents <br>

&nbsp;

* Normalized Discounted Cumulative Gain

As we all know that the order of result matters in the information retrival; The earlier relevant documents show, better the result you get. Let's check the first 3 documents. 

Here is the top 3 for UC 1:
|      |                                                           |           |           | 
|------|-----------------------------------------------------------|-----------|-----------| 
| Id   | Variety                                                   | Score     | Relevant? | 
| 426  | Chow Mein Oyster Sauce BBQ Flavor                         | 10.532438 | 1         | 
| 596  | Quinoa Noodle With Oyster Sauce And Vegetables            | 9.262579  | 1         | 
| 2084 | Tiny Noodle With Oyster Flavor                            | 7.621714  | 1         | 


DCG3 = rel1 + rel2/lg(2) + rel3/lg(3)

= rel(Doc 426) + rel(Doc 596)/1 + rel(Doc 2084)/1.6

= 1 + 1/1 + 1/1.6

= 2.625



Here is the top 3 for UC 2:

|      |                                                                                                  |           |           | 
|------|--------------------------------------------------------------------------------------------------|-----------|-----------| 
| Id   | Variety                                                                                          | Score     | Relevant? | 
| 60   | Hearty Japanese Style Curry Udon                                                                 | 7.2289004 | 1         | 
| 1365 | Hello Kitty Dim Sum Noodle Japanese Curry Flavour                                                | 7.2289004 | 1         | 
| 1389 | Soba Curry Noodles With Japanese Yakisoba Sauce                                                  | 7.2289004 | 1         | 

DCG3 = rel1 + rel2/lg(2) + rel3/lg(3)

= rel(Doc 60) + rel(Doc 1365)/1 + rel(Doc 1389)/1.6

= 1 + 1/1 + 1/1.6

= 2.625




Here is the top 3 for UC 3:

|      |                                                       |       |           |           | 
|------|-------------------------------------------------------|-------|-----------|-----------| 
| Id   | Score                                                 | Style | Variety   | Relevant? | 
| 1365 | Hello Kitty Dim Sum Noodle Japanese Curry Flavour     | Cup   | 8.228901  | 1         | 
| 1389 | Soba Curry Noodles With Japanese Yakisoba Sauce       | Cup   | 8.228901  | 1         | 
| 538  | Hello Kitty Dim Sum Noodle Japanese Soy Sauce Flavour | Cup   | 5.205274  | 0         | 

DCG3 = rel1 + rel2/lg(2) + rel3/lg(3)

= rel(Doc 1365) + rel(Doc 1389)/1 + rel(Doc 538)/1.6

= 1 + 1/1 + 0/1.6

= 2


## 4 Discuss the results

I spend lots of time on mapping; and of cause, I learned from the struggle. For instance, different field datatypes with their parameter, synonym_analyzer and the meaning of customizing the BM25. The documentation on Elasticsearch are really helpful; the explanation is very clear. There are lots of functions and setting on Elasticsearch. I enjoy using this platform. At this project, I explore more on BM25 Algorithm and I like BM25 more than the traditional TF*IDF method. In TFIDF, THE impact of term frequency is always increasing, but it's not necessary true that more frequent more relevant. 
