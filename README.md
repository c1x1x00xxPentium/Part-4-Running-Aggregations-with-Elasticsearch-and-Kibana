# Beginner's Crash Course to Elastic Stack Series
## Part 4: Running Aggregations with Elasticsearch andKibana
Welcome to the Beginner's Crash Course to Elastic Stack!

This repo contains all resources shared during Part 4: Running Aggregations with Elasticsearch and Kibana.

We search for things on a daily basis. Sometimes, we seek to retrieve documents based on the specific criteria(i.e. retrieving songs by our favorite artist). Other times, we seek to gain insight by summarizing our data(i.e. Report on monthly revenue generated by an artist’s concert tour). 

Throughout the Beginner’s Crash Course, we primarily focused on retrieving documents by sending queries to Elasticsearch. This workshop will focus on aggregations, which summarizes your data as metrics, statistics, or other analytics!

By the end of this workshop, you will be able to run:
- metric aggregations
- buckets aggregations
- combined aggregations

## Resources

[Table of Contents: Beginner's Crash Course to Elastic Stack](https://github.com/LisaHJung/Beginners-Crash-Course-to-the-Elastic-Stack-Series) This workshop is a part of the Beginner's Crash Course to Elastic Stack series. Check out this table contents to access all the workshops in the series thus far. This table will continue to get updated as more workshops in the series are released! 

[Free Elastic Cloud Trial](https://ela.st/elastic-beginners)

[Instructions](https://dev.to/lisahjung/beginner-s-guide-to-setting-up-elasticsearch-and-kibana-with-elastic-cloud-1joh) on how to access Elasticsearch and Kibana on Elastic Cloud

[Instructions](https://dev.to/elastic/downloading-elasticsearch-and-kibana-macos-linux-and-windows-1mmo) for downloading Elasticsearch and Kibana

[Presentation]()

[Dataset](https://www.kaggle.com/carrie1/ecommerce-data) from Kaggle used for workshop

[Elastic America Virtual Chapter](https://community.elastic.co/amer-virtual/): Want to attend live workshops? Join the Elastic Americal Virtual Chapter to get the deets!

## Preparing the dataset for aggregations
Often times, the dataset will not be optimal for performing aggregations in its original state. 

For example, the data type of a field has may not be recognized by Elasticsearch or the dataset may contain a value in a field that do not belong in that field and etc. 

Those are exact problems that I ran into while working with this dataset. The following are the requests that I sent to yield the results shared during the workshop. 

Copy and paste these requests into Dev Tools in Kibana and run these queries in order specified below. 

**STEP 1: Create a new index(ecommerce_data) with the following mapping.** 
```
PUT ecommerce_data
{
  "mappings": {
    "meta": {
      "created_by": "ml-file-data-visualizer"
    },
    "properties": {
      "Country": {
        "type": "keyword"
      },
      "CustomerID": {
        "type": "long"
      },
      "Description": {
        "type": "text"
      },
      "InvoiceDate": {
        "type": "date",
        "format": "M/d/yyyy H:m"
      },
      "InvoiceNo": {
        "type": "keyword"
      },
      "Quantity": {
        "type": "long"
      },
      "StockCode": {
        "type": "keyword"
      },
      "UnitPrice": {
        "type": "double"
      }
    }
  }
}
```   
**STEP 2: Reindex the data from original index(source) to the one you just created(destination).**
```
POST _reindex
{
  "source": {
    "index": "name of your original index when you added the data to Elasticsearch"
  },
  "dest": {
    "index": "ecommerce_data"
  }
}
```

**STEP 3: Remove negative values from unit_price field.**

When you explore the minimum unit price in this dataset, you will see that the minimum unit price value is -11062.06. To keep our data simple, I used the delete_by_query API to remove unit prices less than 0. 

```
POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "lte": 0
      }
    }
  }
}
```

**STEP 4:Use delete_by_query to delete all UnitPrice values greater than 500.**

When you explore the maximum unit price in this dataset, you will see that the maximum unit price value is 38970. When the data is manually examined, majority of the unit prices are less than 500. The max value of 38970 would have skewed the average. To simplify our demo, I used the delete_by_query API to remove unit prices greater than 500.
```
POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "gte": 500
      }
    }
  }
}
```
## Review from previous workshops
There are two main ways to search in Elasticsearch:
1) `Queries`retrieve documents that match the specified criteria. 
2) `Aggregations` present the summary of your data as metrics, statistics, and other analytics. 

![image](https://user-images.githubusercontent.com/60980933/113929277-1ad16900-97ad-11eb-8e49-2aacdf0f430b.png)

![image](https://user-images.githubusercontent.com/60980933/113929349-33418380-97ad-11eb-9cc3-f7591ab69f68.png)

### Get information about documents in an index
The following query will retrieve all documents that exist in the specified index. This query is a great way to explore the structure and content of your document. 

Syntax: 
```
GET Enter_name_of_the_index_here/_search
```
Example: 
```
GET ecommerce_data/_search
```
Expected response from Elasticsearch:

Elasticsearch displays a number of hits(line 12) and a sample of 10 search results by default. The first search result(a document)is shown in lines 17-31.  The field "_ source"(line 22) lists all fields or content of the document.

![image](https://user-images.githubusercontent.com/60980933/112375185-9c52d280-8ca8-11eb-9952-16f24171dfbd.png)

## Aggregations Requests
Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "aggs" or "aggregations": {
    "Name your aggregations report here": {
      "Name the metric type here": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```

### Metric Aggregations 
`Metric`aggregations are used to compute numeric values based on your dataset. It can be used to calculate values of `sum`,`min`, `max`, `avg`, and unique count(`cardinality`). 

#### Compute the `sum` of all unit prices in the data set

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "aggs" or "aggregations": {
    "Name your aggregations report here": {
      "sum": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example:
```
GET ecommerce_data/_search
{
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

By default, Elasticsearch will return top 10 most relevant documents (Lines 16+). 

![image](https://user-images.githubusercontent.com/60980933/114756805-5b366700-9d18-11eb-8ea4-1164821e715f.png)

When you minimize hits(red box- line 10), you will see the aggregations report we named sum_unit_price(image below). This report displays the sum of all unit prices listed in our data set. 

![image](https://user-images.githubusercontent.com/60980933/114757167-c122ee80-9d18-11eb-9d7a-3856612c7de0.png)

If the purpose of running an aggregation is solely to get the values of aggregations, you can add the size parameter and set it to 0 as shown below. This parameter will prevent Elasticsearch from fetching the top 10 documents and speed up the query. 

Example:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

Now you do not need to minimize hits to get to the aggregations report! We will be adding the size parameter to all aggregations request from this point on. 

![image](https://user-images.githubusercontent.com/60980933/114758361-1c091580-9d1a-11eb-94df-58afa67e20c4.png)

#### Compute the lowest(`min`) unit price of an item 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "min": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "lowest_unit_price": {
      "min": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

The lowest unit price of an item is 1.01. 
![image](https://user-images.githubusercontent.com/60980933/112509885-869be680-8d56-11eb-9c2e-5935ff7437e8.png)

#### Compute the highest(`max`) unit price of an item 
Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "max": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "highest_unit_price": {
      "max": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:
The highest unit price of an item is 498.79. 

![image](https://user-images.githubusercontent.com/60980933/112511189-cca57a00-8d57-11eb-9ab3-809b2a410636.png)

#### Compute the `average` unit price of items in the inventory 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "avg": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "average_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch: 

Average unit price in our invetory is around 4.39.

![image](https://user-images.githubusercontent.com/60980933/112511759-58b7a180-8d58-11eb-811f-8d6cb852c220.png)

#### `Stats` Aggregation: Compute the count, min, max, avg, sum in one go

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "stats": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "all_stats_unit_price": {
      "stats": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected Response from Elasticsearch:

Stats aggregation will yield the values of `count`(the number of unit prices aggregation was performed on), `min`, `max`, `avg`, and `sum`(sum of all unit prices in the data set). 

![image](https://user-images.githubusercontent.com/60980933/114769078-f20a2000-9d26-11eb-9827-e7672cbba158.png)


#### The Cardinality Aggregations
The cardinality aggregation computes the count of unique values for a given field. 

Syntax:
```
GET Enter_name_of_the_index_here
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "cardinality": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "number_unique_customers": {
      "cardinality": {
        "field": "CustomerID"
      }
    }
  }
}
```
Expected response from Elasticsearch: 

Approximately, there are 4325 unique number of customers in our data set. 
![image](https://user-images.githubusercontent.com/60980933/114774709-aeff7b00-9d2d-11eb-9da5-8faf0dc87292.png)

#### Limiting the scope of an aggregation: Compute the average unit price of items sold in Germany

In previous examples, aggregations were performed on all documents in the ecommerce_data index. What happens if you want to run an aggregation on a subset of the documents? 

For example, our ecommerce_data index contains e-commerce data from multiple countries. What if you want to focus on the average of unit price of items sold in Germany? 

To limit the scope of the aggregation, a query clause can be added to the request. The query clause defines the subset of documents that aggregations should be performed on.  

The combined query and aggregations look like the following: 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "query": {
    "Enter match or match_phrase here": {
      "Enter the name of the field": "Enter the value you are looking for"
    }
  },
  "aggregations": {
    "Name your aggregation here": {
      "Specify aggregation type here": {
        "field": "Name the field you want to aggregate here"
      }
    }
  }
}
```
Example: 
```
GET e_commerce/_search
{
  "size": 0,
  "query": {
    "match": {
      "Country": "Germany"
    }
  },
  "aggs": {
    "germany_average_unit_price": {
      "avg": {
        "field":"UnitPrice"
      }
    }
  }
}
```

Expected response from Elasticsearch:

The average of unit price of items sold in Germany is 4.58.
![image](https://user-images.githubusercontent.com/60980933/112534501-c1ab1380-8d70-11eb-9ce7-507953cc26d0.png)

The combination of query and aggregation allowed us to perform aggregations on a subset of documents. What if we wanted to perform aggreations on several subsets of documents? 

This is where Bucket Aggregations come into play! 

###  Bucket Aggregations
Bucket aggregations group documents into several sets of documents called buckets. 
Buckets are collections of documents that share a common criteria.

The following are different types of bucket aggregations. 

1. Date Histogram Aggregation
2. Histogram Aggregations
3. Range Aggregation
4. Terms aggregations

#### 1.The Date Histogram Aggregation
When you collect data over time (i.e. transaction data over a year), you may be able to gain valuable insights if you can group documents based on a given time interval. 

There are two ways to define the time interval.
1. `Fixed_interval`
2. `Calendar_interval`

1. `Fixed_interval`:The inverval is always constant
Example: Create a bucket for every seven days. 

Syntax:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "fixed_interval": "Specify the interval here"
      }
    }
  }
}
```

Example:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_week": {
      "date_histogram": {
        "field": "InvoiceDate",
        "fixed_interval": "7d"
      }
    }
  }
}
```
Expected response from Elasticsearch:
Elasticsearch creates a bucket for every 7 days and hows the number of documents grouped into each bucket. 

![image](https://user-images.githubusercontent.com/60980933/114788575-f17d8380-9d3e-11eb-90e0-bcee2c7209dd.png)

2. `Calendar_interval`: The interval may vary.
Ex. Create a bucket for each month
Some months have different number of days!

Syntax:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "calendar_interval": "Specify the interval here"
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch creates a bucket for each month and shows the number of documents that fall within the time range. 

![image](https://user-images.githubusercontent.com/60980933/114789926-170b8c80-9d41-11eb-941f-0c6f82311349.png)

#### Histogram Aggregations
`The histogram aggregations` is similar to the `date_histogram` aggregation. However, it can be applied to any numerical field given that your intervals have the same preset size.

Ex. Create a bucket for increasing interval by $5 

Syntax: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs" or "aggregations": {
    "Name your aggregation here": {
      "histogram": {
        "field":"Name the field you want to aggregate on here",
        "interval": Specify the interval here
      }
    }
  }
}
```
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "preset_price_range": {
      "histogram": {
        "field": "UnitPrice",
        "interval": 5
      }
    }
  }
}
```

Expected response from Elasticsearch:

![image](https://user-images.githubusercontent.com/60980933/114792177-107f1400-9d45-11eb-8595-580e524c9b39.png)

### The Range Aggregation 
Histogram aggregations requires that your intervals have the same preset size. If your use case requires that you define your own intervals(i.e. custom intervals or intervals of varying sizes), the `range aggregation` comes in handy!  

For example, what if you wanted to know how many transactions occurred for items whose unit prices between minimum value and $50 or between $50-$200, or greater than $200? 

Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs" or "aggregations": {
   "Name your aggregation here": {
      "range": {
        "field": "Name the field you want to aggregate on here",
        "ranges": [
          {
            "to": x
          },
          {
            "from": x,
            "to": y
          },
          {
            "from": z
          }
        ]
      }
    }
  }
}
````
Example: 
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "custom_price_ranges": {
      "range": {
        "field": "UnitPrice",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 200
          },
          {
            "from": 200
          }
        ]
      }
    }
  }
}
```
Expected response from Elasticsearch:
![image](https://user-images.githubusercontent.com/60980933/114792261-44f2d000-9d45-11eb-9298-6bae6dcf8f06.png)

#### The Terms Aggregation
The terms aggregation creates a new bucket for every unique term it encouters for the specified field. It is often used to find most frequently found terms in a document. 

For example, what if you wanted to identify your 5 customers with highest number of transactions? 
Syntax:
```
GET Enter_name_of_the_index_here/_search
{
  "aggregations": {
    "Name your aggregation here": {
      "terms": {
        "field": "Name the field you want to aggregate on here",
        "size": State how many top results you want returned here
      }
    }
  }
}
```
Example:
```
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5
      }
    }
  }
}
```
Expected response from Elasticsearch:
Elasticsearch will return top 5 customer IDs with the highest number of transaction documents. 
![image](https://user-images.githubusercontent.com/60980933/114796514-6906df00-9d4e-11eb-862e-ac8eed4a10e2.png)

### Combining Aggregations
Some questions need a combination of aggregation to be answered. What is the sum of revenue per day? 
This requires both metric and buckets aggregation. 

#### Daily revenue
```
GET e_commerce_2/_search
{
  "size": 0,
  "aggs": {
    "orders_by_day": {
      "date_histogram": {
        "field":"InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": 
                "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        }
      }
    }
  }
}
```
Split the documents into daily buckets. multiply unitprice and quantity of each document than get the sum of it. To do this, create a new aggregation inside the first aggregation. 
This new aggregation compute the sum of the field unitprice and quantity on every bueckt. 

Each bucket - orders by day
alson contains the sub aggregation called daily revenue

```
GET e_commerce_2/_search
{
  "size": 0,
  "aggs": {
    "orders_by_day": {
      "date_histogram": {
        "field":"InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": 
                "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        }
      }
    }
  }
}
```

Response from Elasticsearch:

#### Calculating multiple metrics per bucket
Caluculate daily revenue also add unique number of customer per day, add cardinality aggregation inside the daily historgram aggregation
```
GET e_commerce_2/_search
{
  "size": 0,
  "aggs": {
    "orders_by_day": {
      "date_histogram": {
        "field":"InvoiceDate",
        "calendar_interval": "day",
        "order": {
          "daily_revenue": "desc"
        }
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "inline":
                "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}
```

#### Creating multiple subuckets
month and country and monthly revenue per country
```
GET e_commerce_2/_search
{
  "size": 0,
  "aggs": {
    "by_country": {
      "terms": {
        "field": "Country",
        "order": {
          "total_revenue": "desc"
        }
      },
        "aggs": {
          "total_revenue": {
            "sum": {
              "script": {
                "inline": "doc['UnitPrice'].value * doc['Quantity'].value"
              }
            }
          },
          "orders_by_month": {
            "date_histogram": {
              "field": "InvoiceDate",
              "calendar_interval": "month"
            },
            "aggs": {
              "monthly_revenue": {
                "sum": {
                  "script": {
                    "inline": "doc['UnitPrice'].value * doc['Quantity'].value"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
```

