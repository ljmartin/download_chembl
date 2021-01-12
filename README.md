# download_chembl
download active ligands associated with a protein target from ChEMBL using web services API

```python
import requests
import json
import pandas as pd

pchembl = 5 #equivalent to 10uM
chembl_accession = 'CHEMBL4333' #S1p receptor
n = 10_000 #try loading n first. If there's more than 10,000 ligands, they will be queried again.

print('Counting bioactivities:')
urlString = "https://www.ebi.ac.uk/chembl/api/data/activity.json?target_chembl_id__exact=%s&pchembl_value__gt=%s&limit=%s" % (chembl_accession, pchembl, n)
webQuery = json.loads(requests.get(urlString).content)
total_count = webQuery['page_meta']['total_count']
print(f'{total_count} ligands listed')


if total_count>n:
    print('Lots of ligands, loading more...')
    urlString = "https://www.ebi.ac.uk/chembl/api/data/activity.json?target_chembl_id__exact=%s&pchembl_value__gt=%s&limit=%s" % (chembl_accession, pchembl, total_count)
    webQuery = json.loads(requests.get(urlString).content)
    print('Done')
    
activities = webQuery['activities']

df = pd.DataFrame(activities)

while len(df)<total_count:
    urlString = "https://www.ebi.ac.uk"+webQuery['page_meta']['next']
    print(urlString)
    webQuery = json.loads(requests.get(urlString).content)
    activities = webQuery['activities']
    df = pd.concat([df, pd.DataFrame(activities)])
    print(f'Loaded {len(df)} of {total_count}')
```
