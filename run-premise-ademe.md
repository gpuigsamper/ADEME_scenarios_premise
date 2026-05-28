---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.16.4
  kernelspec:
    display_name: premise_ademe
    language: python
    name: premise_ademe
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
# It should be ecoinvent 3.9.1 or 3.10.1
ECO_VERSION="3.10"
NAME_BW_PROJECT="ecoinvent_3_10_1"
```

```python
#Open the brightway project
bw2data.projects.set_current(NAME_BW_PROJECT)
#bw2data.projects.current

#Print the databases that are in your project
list(bw2data.databases)
```

```python
if ECO_VERSION=="3.9.1":
    ecoinvent_db_name='ecoinvent-3.9.1-cutoff'
    ecoinvent_db_name="ecoinvent-3.9.1-biosphere"
if ECO_VERSION=="3.10": #Put 3.10 even if you use 3.10.1 version
    ecoinvent_db_name='ecoinvent-3.10.1-cutoff'
    ecoinvent_bio_db_name="ecoinvent-3.10.1-biosphere"
```

```python
#if needed to delete a database
#del bw2data.databases['ei_cutoff_3.10_tiam-ucl_SSP2-RCP19_2050_S1 2026-05-28']
```

# Load input data

```python
fp = r"datapackage.json"
ademe = Package(fp)
```

```python
#INFO : Datapackage relies on 3 resources files called
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
# Other French scenarios are not used in this repository
fr_scenario_2="S2"# - Territorial cooperation"
fr_scenario_3="S3 Renew"# - Green technologies renewables"
fr_scenario_3bis="S3 Nuc"# - Green technologies nuclear"
fr_scenario_4="S4"# - Repairing bet"
```

## Generate the database

```python
scenarios = [
        {"model": model_2, "pathway":world_scenario_2, "year": year, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_2, "pathway":world_scenario_4, "year": year, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        ]
```

```python
ndb = NewDatabase(
        scenarios = scenarios,        
        source_db=ecoinvent_db_name,
        source_version=ECO_VERSION,
        key= ,#to be asked to Romain Sacchi
        biosphere_name=ecoinvent_bio_db_name,
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
ndb.write_db_to_brightway()
```

```python
bw2data.databases
```

## Explore the new databases

```python
db_name='ei_cutoff_3.10_tiam-ucl_SSP2-RCP19_2050_S1 2026-05-28'
```

```python
acts=[act for act in bw2data.Database(db_name) if "Tr2050" in act["name"]]
acts
```

```python
act=[act for act in bw2data.Database(db_name) if act["name"]=="market for electricity, high voltage, Tr2050"][0]
act
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
exc = [exc for exc in act.exchanges()]
exc
#exc = [exc for exc in act.exchanges() if "wind" in e.input["name"]][0]  # ¡¡¡Nota: e.input et torna l'activitat!!!!
```

```python

```
