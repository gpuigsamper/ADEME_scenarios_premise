# Premise + French prospective scenarios : Transition(s) 2050 / ADEME
Implementation of French prospective scenarios from ADEME study "Transition(s) 2050" into ecoinvent database with premise


What does this repository do ?
-----------
![boundaries map](https://github.com/oie-mines-paristech/ADEME_scenarios_premise/blob/main/assets/map.png?raw=true)

This is a repository containing the implementation of prospective scenarios for France into ecoinvent. It generates databases for providing decent living standards (DLS) under the considered policy scenario (S1 narrative) until 2050. The prospective scenarios are provided in the "Transition(s) 2050" study by the French Agency for Ecological Transition - ADEME.   

The scope of ADEME prospective study is metropolitan France, from now to 2050, and covers most of the economic sectors.
This repository creates market-specific activities in the life-cycle inventory database ecoinvent for the following sectors in France:

* Electricity
* Hydrogen
* Gas
* Biomethane and synthetic natural gas
* Biogas
* Liquefied petroleum gas (LPGs)
* Liquid carburants
* Heat
* End-of-life treatment

Demand levels are set to decent living standards levels, thereby generating consuming markets for the different DLS dimensions:

* Clothing
* Collective services
* Communication
* Education
* Healthcare
* Hygiene
* Nutrition
* Mobility
* Shelter

The evolution at the world regional scale are modeled by coupling the French scenarios with a global scenario provided by integrated assessment models (IAM).


ADEME prospective study 
------------------------

Prospective scenarios are extracted from the study : Transition(s) 2050, ADEME, 2021\
[`Website`](https://www.ademe.fr/les-futurs-en-transition/) \
[`Full Report`](https://librairie.ademe.fr/recherche-et-innovation/5072-prospective-transitions-2050-rapport.html) \
[`Data repository`](https://data-transitions2050.ademe.fr/)

ADEME provides 5 scenarios : 
* S1 : Frugal generation (Génération Frugale) > The transition is driven mostly by sobriety and constraint.
* S2 : Territorial cooperation (Coopération territoriale) > The society transformation is based on a shared governance and on territorialization strategies.
* S3 Renew : Green technologies based on renewables development (Technologies vertes) > The transition is based on innovation and development of low carbon technologies, especially renewable energies.  
* S3 Nuc : Green technologies based on nuclear development (Technologies vertes) > The transition is based on innovation and development of low carbon technologies, especially nuclear energy.  
* S4 : Repairing bet (Pari réparateur) > The transition relies highly on new technologies development, without any society lifestyle changes. 

Informations about scenarios : [`here in French`](https://www.ademe.fr/les-futurs-en-transition/les-scenarios/)

How is the repository organized ?
-----------

This repository is meant to be used with the open-source python library [`premise`](https://github.com/polca/premise), using the [`user-defined scenario functionnality`](https://premise.readthedocs.io/en/latest/user_scenarios.html).
The data relating to the annual production volumes for each scenario have been formatted and organised in a data package defined by the Frictionless standards (Walsh and Pollock, 2022). This data package is read and interpreted by `premise`. We therefore store a number of scenarios in a single data package.

This datapackage contains four files necessary to the scenarios implementation into the ecoinvent LCA database: 

* A **datapackage.json** file, which provides the metadata for the data package (e.g. authors, scenario descriptions, list and locations of resources, etc.). 
* A **config.yaml** file which provides the correspondence between the scenario variables and the LCA datasets in the ecoinvent DB, as well as the additional "LCA datasets" when they are not available in the ecoinvent database. 
* A tabular data file **scenario_data.xlsx** containing the time series for each variable in the set of scenarios. 
* An optional Excel file **LCI-Tr2050.xlsx** containing the LCA inventories of the additional "LCA datasets" for any technology not initially present in the ecoinvent database. 

Additionally, a pdf document called "supplementary information" presents the methodological choices that where made to build this model.


How to use this notebook ?
------------------
* 0. Prerequisites: ecoinvent licence
* 1. Install the environment as explained [`here`](https://github.com/polca/premise?tab=readme-ov-file#how-to-install-this-package).
  Use premise version => 3.2.4
* 2. Create a brightway project and load ecoinvent database in the project. It can be done using [`ecoinvent_interface`](https://github.com/brightway-lca/ecoinvent_interface).
* 3. Run the following script for a chosen combination of Year x IAM model x IAM scenario x French scenario. Here is an example for two French scenarios combined with the same IAM scenario, with ecoinvent 3.10.1.
* 3. (bis) Or run the file run-premise-ademe.md. Example notebook to run premise with and without external scenarios [`here`](https://github.com/polca/premise/tree/master/examples).

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
    #Choose the Year
    year=2050
    #Choose the French scenario 
    fr_scenario_1="S1"
    
    scenarios = [
        {"model": model_1, "pathway":world_scenario_1, "year": year, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        {"model": model_1, "pathway":world_scenario_2, "year": year, "external scenarios": [{"scenario": fr_scenario_1, "data": ademe}]},
        ]
  
    ndb = NewDatabase(
        scenarios = scenarios,        
        source_db=ecoinvent_3_10_db_name,
        source_version="3.10",
        key= , #ask the key to Romain Sacchi
        biosphere_name=ecoinvent_3_10_bio_db_name,
        )
  
    ndb.update()
  
    ndb.write_db_to_brightway()
  
    list(bd.databases)

  ```
  
A prospective version of ecoinvent is generated for each combination of : Year x IAM model x IAM scenario x French scenario.
The newly created market datasets are tagged with 'Tr2050', for example : `market for electricity, high voltage, Tr2050` (FR) or `market for nutrition DLS, Tr2050` (FR)

Ecoinvent database compatibility
--------------------------------
ecoinvent 3.10.1 cut-off

IAM scenario compatibility
---------------------------
The user can couple each French scenario with a global scenario (IAM) provided by premise.\
The available IAM scenarios provided by premise can be explored [`here`](https://premisedash-6f5a0259c487.herokuapp.com/)\
The choice of IAM scenario is under the responsability of the user of this repository.

Authors of this data package
----------------------------
* Joanna Schlesinger-Martinat
* Gonzalo Puig-Samper

Acknowledgements
----------------------------
We would like to thank ADEME experts for providing datasets and explanations to understand scenarios and datasets, especially Jean-Michel Parrouffe for the multiple discussions.

Funding
-------
This work is supported by the ADEME agency, in the context of
the [`HYSPI project`](https://www.psi.ch/en/ta/projects/hyspi) and by ENGIE in the context of Gonzalo Puig-Sampers' PhD (CIFRE individual fellowship [grant number 2022/0710]).









