# New York City Parking Violations
### Project Description
In this project, I applied what I have learned about EC2, Terminal, Docker, Elasticsearch and Kibana to a real-world dataset powered by NYC Open Data. This dataset has 14.4 million rows and 16 columns. Each row is an open parking and camera violations issued in New York city. The columns include the summons number, violation time, violation type, penalty amount, and other violation details.

I wrote a python script that runs in docker to consume data from the Socrata Open Data API and then pushes that information into an Elasticsearch cluster provisioned via AWS. After loading all the data into an Elasticsearch instance, I visualized and analyzed the data with Kibana.



### Technology/Framework used
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

#### In Dockerfile
```python
# We want to go from the base image of python:3.9
FROM python:3.9

# This is the equivalent of “cd /app” from our host machine
WORKDIR /app

# Let’s copy everything into /app
COPY .  /app

# Installs thedependencies. Passes in a text file.
RUN pip install -r requirements.txt

# This will run when we run our docker container
ENTRYPOINT ["python", "src/main.py"]
```

#### In requirements.txt
```python
sodapy==2.1.0
requests==2.26.0
```

#### In main.py
```python
from sodapy import Socrata
import requests
from requests.auth import HTTPBasicAuth
import argparse
import sys
import os
import json

parser = argparse.ArgumentParser(description='Process data from NYC parking violation.')
parser.add_argument('--page_size', type=int, help='how many rows to get per page', required=True)
parser.add_argument('--num_pages', type=int, help='how many pages to get in total')
args = parser.parse_args(sys.argv[1:])
print(args)

DATASET_ID = os.environ["DATASET_ID"]
APP_TOKEN = os.environ["APP_TOKEN"]
ES_HOST = os.environ["ES_HOST"]
ES_USERNAME = os.environ["ES_USERNAME"]
ES_PASSWORD = os.environ["ES_PASSWORD"]
INDEX_NAME = os.environ["INDEX_NAME"]

if __name__ == '__main__':

    try:
        resp = requests.put(f"{ES_HOST}/{INDEX_NAME}", auth=HTTPBasicAuth(ES_USERNAME, ES_PASSWORD),
                            json={
                                "settings": {
                                    "number_of_shards": 1,
                                    "number_of_replicas": 1
                                },
                                "mappings": {
                                    "properties": {
                                        "plate": {"type": "keyword"},
                                        "state": {"type": "keyword"},
                                        "license_type": {"type": "keyword"},
                                        "summons_number": {"type": "keyword"},
                                        "issue_date": {"type": "date", "format": "mm/dd/yyyy"},
                                        "violation_time": {"type": "keyword"},
                                        "violation": {"type": "keyword"},
                                        "fine_amount": {"type": "float"},
                                        "penalty_amount": {"type": "float"},
                                        "interest_amount": {"type": "float"},
                                        "reduction_amount": {"type": "float"},
                                        "payment_amount": {"type": "float"},
                                        "amount_due": {"type": "float"},
                                        "precinct": {"type": "keyword"},
                                        "county": {"type": "keyword"},
                                        "issuing_agency": {"type": "keyword"}
                                    }
                                },
                            }
                            )

        resp.raise_for_status()
        print(resp.json())
    except Exception as e:
        print("Index already exists! Skipping")
    

    for page in range(0, args.num_pages):
        es_rows=[]
        rows = Socrata("data.cityofnewyork.us", APP_TOKEN).get(DATASET_ID, order = "summons_number", limit=args.page_size, offset= page* (args.page_size))


        for row in rows:
            try:
                # Convert
                es_row = {}
                es_row["plate"] = row["plate"]
                es_row["state"] = row["state"]
                es_row["license_type"] = row["license_type"]
                es_row["summons_number"] = row["summons_number"]
                es_row["issue_date"] = row["issue_date"]
                es_row["violation_time"] = row["violation_time"]
                es_row["violation"] = row["violation"]
                es_row["fine_amount"] = float(row["fine_amount"])
                es_row["penalty_amount"] = float(row["penalty_amount"])
                es_row["interest_amount"] = float(row["interest_amount"])
                es_row["reduction_amount"] = float(row["reduction_amount"])
                es_row["payment_amount"] = float(row["payment_amount"])
                es_row["amount_due"] = float(row["amount_due"])
                es_row["precinct"] = row["precinct"]
                es_row["county"] = row["county"]
                es_row["issuing_agency"] = row["issuing_agency"]

            except Exception as e:
                #print(f"Error!: {e}, skipping row: {row}")
                continue

            es_rows.append(es_row)


# bulk api in first loop
        bulk_upload_data = ""
        for line in es_rows:
            print(f'Handling row {line["summons_number"]}')
            action = '{"index": {"_index": "' + INDEX_NAME + '", "_type": "_doc", "_id": "' + line[
                "summons_number"] + '"}}'
            data = json.dumps(line)
            bulk_upload_data += f"{action}\n"
            bulk_upload_data += f"{data}\n"

        try:
            resp = requests.post(f"{ES_HOST}/_bulk",
                                 data=bulk_upload_data, auth=HTTPBasicAuth(ES_USERNAME, ES_PASSWORD),
                                 headers={"Content-Type": "application/x-ndjson"})
            resp.raise_for_status()
            print('Done')

        except Exception as e:
            print(f"Failed to insert in ES: {e}, skipping row: {row}")

```


**Step4**: Visualizing and Analysis on Kibana (OpenSearch Dashboard)

![Alt text](https://github.com/jinote/my-projects/blob/main/BigDataTech/Kibana%20Dashboard.jpg)


**Observation:**
1. In photo school zone speed violation points, parking violations have occured the most for the past 10 years (2012-2021)
2. Parking violations mostly occured in the morning. 11:30AM was the time when it occured most.  
3. New York has the highest fine amounts in 2020-2021. 
4. Unlike average fine amounts, interest, penalty, and reduction amounts have decreased in the past 6 years. 





