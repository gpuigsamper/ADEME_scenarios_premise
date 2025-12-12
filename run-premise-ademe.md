---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.4
  kernelspec:
    display_name: premise5
    language: python
    name: premise5
---

```python
from premise import *
import bw2data
import bw2io
from datapackage import Package
```

# Open brightway project

```python
#Put the name of your brightway project
# ecoinvent + biosphere shall be already loaded as databases of the project
# It should be ecoinvent 3.9 or more recent version
NAME_BW_PROJECT="HySPI_premise_Tr2050_7"
```

```python
#Open the brightway project
bw2data.projects.set_current(NAME_BW_PROJECT)
#bw2data.projects.current

#Print the databases that are in your project
list(bw2data.databases)
```

```python
ecoinvent_3_9_db_name='ecoinvent-3.9.1-cutoff'
ecoinvent_3_9_bio_db_name="ecoinvent-3.9.1-biosphere"
```

```python
#if needed to delete a database
#del bw2data.databases['tiam-SSP2-Base-N1']
```

# Load input data

```python
fp = r"datapackage.json"
ademe = Package(fp)
```

```python
#Datapackage relies on 3 resources files called
ademe.resource_names
```

```python
# Choose the scenario to generate
#IAM model
model_1="image"
model_2="tiam-ucl"
model_3="remind"

#world scenario

world_scenario_1="SSP2-Base"
world_scenario_2="SSP2-RCP45"
world_scenario_3="SSP2-RCP26"
world_scenario_4="SSP2-RCP19"
world_scenario_5="SSP2-NPi"

#Year
year=2050

#French scenario
fr_scenario_1="S1"# - Frugal generation"
fr_scenario_2="S2"# - Territorial cooperation"
fr_scenario_3="S3 Renew"# - Green technologies renewables"
fr_scenario_3bis="S3 Nuc"# - Green technologies nuclear"
fr_scenario_4="S4"# - Repairing bet"
```

## For tests : exploration of one database 


### Generate the database

```python
scenarios = [
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_2, "data": ademe}]},
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_3, "data": ademe}]},
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_3bis, "data": ademe}]},
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_4, "data": ademe}]},

        ]
```

```python
ndb = NewDatabase(
        scenarios = scenarios,        
        source_db=ecoinvent_3_9_db_name,
        source_version="3.9.1",
        key='tUePmX_S5B8ieZkkM7WUU2CnO8SmShwmAeWK9x2rTFo=',
        biosphere_name=ecoinvent_3_9_bio_db_name,
        #use_multiprocessing=True
)
```

```python
#This updates all the markets (included in premise) according to IAM scenarios chosen + the external French scenario
ndb.update() 

#This updates only the external French scenario. It does not work as we need updated European market for electricity 
#to model the imports for French Tr20250 markets.
#ndb.update(["external"]) 

#This updates electricity markets according to IAM scenarios + the external French scenario
#ndb.update(["electricity","external"])
```

```python
#ndb.write_superstructure_db_to_brightway()
```

```python
#Not mandatory for tests
ndb.write_db_to_brightway()
#ndb.write_db_to_brightway(name=['tiam - rcp45 - S2 - last et fossil'])
```

```python
bw2data.databases
```

## Explore nbd without printing the database to brightway (to save time) 

```python
#
clear_cache() #if too slow
```

```python
ndb.scenarios[0].keys()
```

```python
list_act=ndb.database
```

```python
keyword="pumped storage"

for act in list_act:
    if keyword in act["name"]:    
        print(act["name"])
```

```python
ndb.database[0]["name"]
```

```python
# Choose the scenario number to be explored
n_scenario=1

list_act=ndb.scenarios[n_scenario-1]["database"]
```

```python
#Print all the activities that contain a keyword
keyword="Tr2050"

for act in list_act:
    if keyword in act["name"]:    
        print(act["name"])
```

```python
activity_name = "electricity production, hydro, pumped storage, Tr2050"

#activity_name = "market for hydrogen, gaseous, for biofuel refinery use, Tr2050"

for act in list_act:
    if activity_name==act["name"]:    
        print(act["location"])
```

```python
#Print all exchanges of a given activity

for act in list_act:
    if activity_name==act["name"]:    
        for e in act["exchanges"]:
            amount=e["amount"]
            print(f"{amount:.2f}","|", e["unit"],"|", e["name"] ) 
```

```python
#Print all technosphere exchanges of a given activity
for act in list_act:
    if activity_name==act["name"] and act["location"]=='FR':    
        for e in act["exchanges"]:
            amount=e["amount"]
            if e['type']=='technosphere':
                print(f"{amount:.2f}","|", e["unit"],"|", e["name"],"|", e["location"] ) 
```

### Explanations (can be skipped)

```python
#ndb.scenarios is a list. Its length equals the number of scenarios.
type(ndb.scenarios), len(ndb.scenarios)
```

```python
#ndb.scenarios[0] refers to the first scenario extracted.
ndb.scenarios[0].keys()
```

```python
#ndb.scenarios[0]["database"] is the list of all the activities (dict) contained in the generated database
list_act=ndb.scenarios[0]["database"]
act_test=list_act[0]
type(list_act),len(list_act),act_test.keys()
```

```python
ndb.scenarios[0]
```

```python
#List of the keys of an exchange
act_test["exchanges"][0].keys()
```

### Explore the new databases

```python
db_name='ei_cutoff_3.9_tiam-ucl_SSP2-RCP45_2050_S4 2025-03-12'
```

```python
acts=[act for act in bw2data.Database(db_name) if "Tr2050" in act["name"]]
```

```python
acts

```

```python
act=[act for act in bw2data.Database(db_name) if "market for electricity, high voltage, Tr2050" in act["name"] and act["location"]=="FR"][0]
act
```

```python
act=[act for act in bw2data.Database(db_name) if "empty" in act["name"]][0]
act
```

```python
exc = [exc for exc in act.exchanges()]
exc
#exc = [exc for exc in act.exchanges() if "wind" in e.input["name"]][0]  # ¡¡¡Nota: e.input et torna l'activitat!!!!
```

```python
exc[0].input
```

```python
climate = ('EF v3.1', 'climate change', 'global warming potential (GWP100)')
#impact calculation
lca = act.lca(method=climate, amount=1)
score = lca.score
unit = bw2data.Method(climate).metadata["unit"]
score
```

```python

```

```python

```
