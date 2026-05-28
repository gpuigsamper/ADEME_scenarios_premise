# French prospective scenarios *Transition(s) 2050* 
Implementation of the S1 narrative from French prospective scenarios *Transition(s) 2050* from the French Ecological Transition Agency (ADEME) into the ecoinvent database.

Publication
------------------------

This data package is used to produce the results of the following publication:
**Decent living within planetary boundaries? A methodological framework for assessing prospective policy scenarios**
Gonzalo Puig-Samper, Joanna Schlesinger-Martinat, Natacha Gondran, Julie Clavreul, Anne Prieur-Vernat, Mikołaj Owsianiak. (*Submitted*)

What does this repository do ?
-----------
This is a repository containing a data package that implements the narratives of *Transition(s) 2050* prospective scenarios for France into ecoinvent. The **S1 narrative** projections are implemented along with a demand based on the fulfilment of decent living standards (DLS) in France.

This data package is built using [`premise`](https://github.com/polca/premise) and can be coupled to global scenarios from integrated assessment models (IAM) in [`premise`](https://github.com/polca/premise) to capture projections outside the scope of *Transition(s) 2050* scenarios. The data package can be used to estimate the environmental impacts of meeting DLS in France under the S1 narrative projections and to estimate the environmental impacts of production pathways and markets, as outlined in the S1 narrative. The data package contains all the necessary files to implement the scenario in ecoinvent database thanks to [`premise`](https://github.com/polca/premise).

![boundaries map](https://github.com/oie-mines-paristech/ADEME_scenarios_premise/blob/main/assets/map.png?raw=true)

The scope of ADEME prospective study is metropolitan France, from now to 2050, and covers most of the economic sectors. This repository creates market-specific activities in the ecoivent life-cycle inventory database for the following sectors in France:

* Electricity
* Hydrogen
* Gas
* Biomethane and synthetic natural gas
* Biogas
* Liquefied petroleum gas (LPGs)
* Liquid fuels
* Heat
* End-of-life treatments

Demand levels are set to DLS levels, thereby generating consuming markets for the different DLS dimensions:

* Clothing
* Collective services
* Communication
* Education
* Healthcare
* Hygiene
* Nutrition
* Mobility
* Shelter

ADEME prospective study 
------------------------

Prospective scenarios are extracted from the study : Transition(s) 2050, ADEME, 2021\
[`Website`](https://www.ademe.fr/les-futurs-en-transition/) \
[`Full Report`](https://librairie.ademe.fr/recherche-et-innovation/5072-prospective-transitions-2050-rapport.html) \
[`Data repository`](https://data-transitions2050.ademe.fr/)
Informations about scenarios : [`here in French`](https://www.ademe.fr/les-futurs-en-transition/les-scenarios/)

ADEME provides 5 scenarios S1, S2, S3 Renew, S3 Nuc, S4. This repository models only databases for S1 scenario, Frugal generation (Génération Frugale). In this scenario, the transition is driven mostly by sobriety and constraint. 

See the [`initial repository`](https://github.com/oie-mines-paristech/ADEME_scenarios_premise) for generating prospective databases according to the five ADEME scenarios (limited to energy sector). 


How is the repository organized ?
-----------

This repository is meant to be used with the open-source python library [`premise`](https://github.com/polca/premise), using the [`user-defined scenario functionnality`](https://premise.readthedocs.io/en/latest/user_scenarios.html).

This datapackage contains four files necessary for the scenarios implementation into the ecoinvent LCA database: 

* A **datapackage.json** file, which provides the metadata for the data package (e.g. authors, scenario descriptions, list and locations of resources, etc.). 
* A **config.yaml** file which provides the correspondence between the scenario variables and the LCA datasets in the ecoinvent database, as well as the additional life-cycle inventories (LCI) when they are not available in the ecoinvent database. 
* A tabular data file **scenario_data.xlsx** containing the time series for each variable in the set of scenarios. 
* An optional Excel file **LCI-Tr2050.xlsx** containing the additional LCIs for any activity not initially present in the ecoinvent database.
  
Branches and Ecoinvent database compatibility
--------------------------------
* ecoinvent 3.9.1 cut-off (branch 'DLS_S1_ecoinvent3.9.1')
* ecoinvent 3.10.1 cut-off (branch 'DLS_S1_ecoinvent3.10')

The branch 'initial_fork' correspond to the [`initial repository`](https://github.com/oie-mines-paristech/ADEME_scenarios_premise) that models the five ADEME scenarios but only for energy sector, only for ecoinvent 3.9.1 and with an older version of premise. 

How to use it ?
------------------
0. Prerequisites: ecoinvent licence
1. With a dedicated environment, create a brightway project and load ecoinvent database in the project. It can be done using [`ecoinvent_interface`](https://github.com/brightway-lca/ecoinvent_interface).
2. Install the environment dedicated to prospective database generation.
```python
conda create -n name_env python=3.11
pip install -r requirements.txt
```
The script is compatible with **premise version 2.3.5** and with **Brightway2** framework. It could be compatible with more recent versions of premise but the authors can not guarantee it. 

3. Generate the prospective databases : run the following script for a chosen combination of Year x IAM model x IAM scenario x French scenario. Here is an example for one French scenario combined with two different IAM scenarios for 2030 and 2050. You can alternatively run the notebook 'run-premise-ademe'. 

  ```python

    from premise import *
    import bw2data as bd
    import bw2io
    from datapackage import Package

    fp = r"datapackage.json"
    ademe = Package(fp)

    NAME_BW_PROJECT="name_of_my_project"
    ecoinvent_3_10_db_name='ecoinvent-3.10.1-cutoff'
    ecoinvent_3_10_bio_db_name="ecoinvent-3.10.1-biosphere"
    #Open the brightway project
    bd.projects.set_current(NAME_BW_PROJECT)
  
    #Choose the IAM model
    model_1="image"
    #Choose the world scenario
    world_scenario_1="SSP2-M"
    world_scenario_2="SSP2-VLHO"
    #Choose the French scenario 
    fr_scenario_1="S1"
    
    scenarios = [
        {"model": model_1, "pathway":world_scenario_1, "year": 2030, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_1, "pathway":world_scenario_1, "year": 2050, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_1, "pathway":world_scenario_2, "year": 2030, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_1, "pathway":world_scenario_2, "year": 2050, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]}
        ]
  
    ndb = NewDatabase(
        scenarios = scenarios,        
        source_db=ecoinvent_3_10_db_name,
        source_version="3.10",
        key= , #ask the key to Romain Sacchi
        biosphere_name=ecoinvent_3_10_bio_db_name,
        )
  
    ndb.update() #nb.update(["external"]) if coupling with IAMs is not desired
  
    ndb.write_db_to_brightway()
  
    list(bd.databases)

  ```
  
A prospective version of ecoinvent is generated for each combination of : Year x IAM model x IAM scenario x French scenario. For more information about IAM scenarios available in premise, please check [`premise documentation`](https://premise.readthedocs.io/en/latest/introduction.html#choosing-the-right-iam).

Databases can be alternatively written as a superstructure database to be used in Activity Browser. 

The newly created market datasets are tagged with 'Tr2050', for example : `market for electricity, high voltage, Tr2050` (FR) or `market for nutrition DLS, Tr2050` (FR).

This repository is only dedicated to the generation of prospective databases. Environmental impacts using the control variables from the planetary boundaries framework (as it is made in the publication) can be calculated using the [`PB-LCIA`](https://github.com/gpuigsamper/PB-LCIA/tree/ei_310) python package.


Authors of this data package
----------------------------
* Gonzalo Puig-Samper
* Joanna Schlesinger-Martinat

Acknowledgements
----------------------------
We would like to thank ADEME experts for providing datasets and explanations to understand scenarios and datasets, especially Jean-Michel Parrouffe for the multiple discussions.

Funding
-------
This work has been supported by the ADEME agency, in the context of
the [`HYSPI project`](https://www.psi.ch/en/ta/projects/hyspi) [nr. 2197D0085] and by ENGIE in the context of Gonzalo Puig-Sampers' PhD (CIFRE individual fellowship [grant number 2022/0710]).
