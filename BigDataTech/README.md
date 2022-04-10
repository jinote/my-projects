# New York City Parking Violations
### Project Description
In this project, the NYC Open Parking and Camera Violation Dashboard, I applied what I have learned about EC2, Terminal, Docker, Elasticsearch and Kibana to a real-world dataset powered by NYC Open Data. This dataset has 1.9 million rows and 19 columns. Each row is an open parking and camera violations issued in New York city traced back from 2016 to now. The columns include the violation type, violation time, fine amount, and penalty amount among other violation details.

I wrote a python script that runs in docker to consume data from the Socrata Open Data API and then pushes that information into an Elasticsearch cluster provisioned via AWS. After loading all the data into an Elasticsearch instance, I visualized and analyzed the data with Kibana.



### Technoloy/Framework used
- Docker
- Service: Elasticsearch, Kibana
- Python: sodapy, sys, argparse, json, requests, os

### Usage
**Step1**: Create an Elasticsearch Domain in AWS

**Step2**: Create an APP Token for the NYC Open Data API
Data link: https://dev.socrata.com/foundry/data.cityofnewyork.us/nc67-uf89


**Step3**: Python Scripting<br>
Some key arguments that I used:
- APP_TOKEN: This is how a user can pass along an APP_TOKEN for the API in a safe manner.
- bigdataproject1:1.0 This is the name of my docker image. 
- ES_HOST: This is the domain endpoint of my Elasticsearch cluster.
- page_size (required): This command line argument is required. It asks for how many records to request from the API per call. 
- num_pages (optional): This command line argument is optional. If not provided, my script should continue requesting data until the entirety of the content has been exhausted. If this argument is provided, continue querying for data num_pages times. 

```python

```



**Step4**: Visualizing and Analysis on Kibana (OpenSearch Dashboard)




