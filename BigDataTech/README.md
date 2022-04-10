# New York City Parking Violations
### Project Description
In this project, the NYC Open Parking and Camera Violation Dashboard, I applied what I have learned about EC2, Docker, Elasticsearch and Kibana to a real-world dataset powered by NYC Open Data. This dataset has 56.9 million rows and 19 columns. Each row is an open parking and camera violations issued in New York city traced back from 2016 to now. The columns include the violation type, the violation time, the fine amount, and the penalty amount among other violation details.

I wrote a python script that runs in docker to consume data from the Socrata Open Data API and then pushes that information into an Elasticsearch cluster provisioned via AWS. After loading all the data into an Elasticsearch instance, I visualized and analyzed the data with Kibana.



### Technoloy/Framework used
- Docker
- Service: Elasticsearch, Kibana
- Python: sodapy, sys, argparse, json, requests, os

### Usage
Step1:

Step2

Step3:

Step4: Visualization in Kibana



