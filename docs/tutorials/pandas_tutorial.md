# Example Notebook: BioCypher and Pandas

<div class="alert alert-info">
**Tip:** You can __[run the tutorial interactively in Google Colab](https://colab.research.google.com/github/biocypher/biocypher/blob/main/docs/notebooks/pandas_tutorial.ipynb)__.
</div>

## Introduction

The main purpose of BioCypher is to facilitate the pre-processing of biomedical data, and thus save development time in the maintenance of curated knowledge graphs, while allowing simple and efficient creation of task-specific lightweight knowledge graphs in a user-friendly and biology-centric fashion.

We are going to use a toy example to familiarise the user with the basic functionality of BioCypher. One central task of BioCypher is the harmonisation of dissimilar datasets describing the same entities. Thus, in this example, the input data - which in the real-world use case could come from any type of interface - are represented by simulated data containing some examples of differently formatted biomedical entities such as proteins and their interactions.

There are two other versions of this tutorial, which only differ in the output format. The first uses a CSV output format to write files suitable for Neo4j admin import, and the second creates an in-memory collection of Pandas dataframes. You can find the former in the tutorial directory of the BioCypher repository. This tutorial simply takes the latter, in-memory approach to a Jupyter notebook.

While BioCypher was designed as a graph-focused framework, due to commonalities in bioinformatics workflows, BioCypher also supports Pandas DataFrames. This allows integration with methods that use tabular data, such as machine learning and statistical analysis, for instance in the [scVerse framework](https://scverse.org).

## Setup

To run this tutorial interactively, you will first need to install perform some setup steps specific to running on Google Colab. You can collapse this section and run the setup steps with one click, as they are not required for the explanation of BioCyper's functionality. You can of course also run the steps one by one, if you want to see what is happening. The real tutorial starts with [section 1, "Adding data"](https://biocypher.org/notebooks/pandas_tutorial.html#Section-1:-Adding-data) (do not follow this link on colab, as you will be taken back to the website; please scroll down instead).


```python
!pip install biocypher
```

### Tutorial files

In the `biocypher` root directory, you will find a `tutorial` directory with
the files for this tutorial. The `data_generator.py` file contains the
simulated data generation code, and the other files, specifically the `.yaml` files, are named according to the
tutorial step they are used in.

Let's download these:


```python
import yaml
import requests
import subprocess

schema_path = "https://raw.githubusercontent.com/biocypher/biocypher/main/tutorial/"
```


```python
!wget -O data_generator.py "https://github.com/biocypher/biocypher/raw/main/tutorial/data_generator.py"
```


```python
owner = "biocypher"
repo = "biocypher"
path = "tutorial"  # The path within the repository (optional, leave empty for the root directory)
github_url = "https://api.github.com/repos/{owner}/{repo}/contents/{path}"

api_url = github_url.format(owner=owner, repo=repo, path=path)
response = requests.get(api_url)

# Get list of yaml files from the repo
files = response.json()
yamls = []
for file in files:
    if file["type"] == "file":
        if file["name"].endswith(".yaml"):
            yamls.append(file["name"])

# wget all yaml files
for yaml in yamls:
    url_path = schema_path + yaml
    subprocess.run(["wget", url_path])
```

Let's also define functions with which we can visualize those


```python
# helper function to print yaml files
import yaml
def print_yaml(file_path):
    with open(file_path, 'r') as file:
        yaml_data = yaml.safe_load(file)

    print("--------------")
    print(yaml.dump(yaml_data, sort_keys=False, indent=4))
    print("--------------")
```

## Configuration
BioCypher is configured using a YAML file; it comes with a default (which you
can see in the
[Configuration](https://biocypher.org/installation.html#configuration) section).
You can use it, for instance, to select an output format, the output directory,
separators, logging level, and other options. For this tutorial, we will use a
dedicated configuration file for each of the steps. The configuration files are
located in the `tutorial` directory, and are called using the
`biocypher_config_path` argument at instantiation of the BioCypher interface.
For more information, see also the [Quickstart
Configuration](https://biocypher.org/quickstart.html#the-biocypher-configuration-yaml-file)
section.

## Section 1: Adding data

### Input data stream ("adapter")
The basic operation of adding data to the knowledge graph requires two
components: an input stream of data (which we call adapter) and a configuration
for the resulting desired output (the schema configuration). The former will be
simulated by calling the `Protein` class of our data generator 10 times.


```python
# create a list of proteins to be imported
from data_generator import Protein
n_proteins = 3
proteins = [Protein() for _ in range(n_proteins)]
```

Each protein in our simulated data has a UniProt ID, a label
("uniprot_protein"), and a dictionary of properties describing it. This is -
purely by coincidence - very close to the input BioCypher expects (for nodes):
- a unique identifier
- an input label (to allow mapping to the ontology, see the second step below)
- a dictionary of further properties (which can be empty)

These should be presented to BioCypher in the form of a tuple. To achieve this
representation, we can use a generator function that iterates through our
simulated input data and, for each entity, forms the corresponding tuple. The
use of a generator allows for efficient streaming of larger datasets where
required.


```python
def node_generator(proteins):
    for protein in proteins:
        yield (
            protein.get_id(),
            protein.get_label(),
            protein.get_properties(),
        )
entities = node_generator(proteins)
```

The concept of an adapter can become arbitrarily complex and involve
programmatic access to databases, API requests, asynchronous queries, context
managers, and other complicating factors. However, it always boils down to
providing the BioCypher driver with a collection of tuples, one for each entity
in the input data. For more info, see the section on
[Adapters](../adapters.md).

As descibed above, *nodes* possess:

- a mandatory ID,
- a mandatory label, and
- a property dictionary,

while *edges* possess:

- an (optional) ID,
- two mandatory IDs for source and target,
- a mandatory label, and
- a property dictionary.

How these entities are mapped to the ontological hierarchy underlying a
BioCypher graph is determined by their mandatory labels, which connect the input
data stream to the schema configuration. This we will see in the following
section.

### Schema configuration
How each BioCypher graph is structured is determined by the schema configuration
YAML file that is given to the BioCypher interface. This also serves to ground
the entities of the graph in the biomedical realm by using an ontological
hierarchy. In this tutorial, we refer to the Biolink model as the general
backbone of our ontological hierarchy. The basic premise of the schema
configuration YAML file is that each component of the desired knowledge graph
output should be configured here; if (and only if) an entity is represented in
the schema configuration *and* is present in the input data stream, will it be
part of our knowledge graph.

In our case, since we only import proteins, we only require few lines of
configuration:


```python
print_yaml('01_schema_config.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id: uniprot
        input_label: uniprot_protein
    
    --------------


The first line (`protein`) identifies our entity and connects to the ontological
backbone; here we define the first class to be represented in the graph. In the
configuration YAML, we represent entities — similar to the internal
representation of Biolink — in lower sentence case (e.g., "small molecule").
Conversely, for class names, in file names, and property graph labels, we use
PascalCase instead (e.g., "SmallMolecule") to avoid issues with handling spaces.
The transformation is done by BioCypher internally. BioCypher does not strictly
enforce the entities allowed in this class definition; in fact, we provide
[several methods of extending the existing ontological backbone *ad hoc* by
providing custom inheritance or hybridising
ontologies](https://biocypher.org/tutorial-ontology.html#model-extensions).
However, every entity should at some point be connected to the underlying
ontology, otherwise the multiple hierarchical labels will not be populated.
Following this first line are three indented values of the protein class.

The second line (`represented_as`) tells BioCypher in which way each entity
should be represented in the graph; the only options are `node` and `edge`.
Representation as an edge is only possible when source and target IDs are
provided in the input data stream. Conversely, relationships can be represented
as both `node` or `edge`, depending on the desired output. When a relationship
should be represented as a node, i.e., "reified", BioCypher takes care to create
a set of two edges and a node in place of the relationship. This is useful when
we want to connect the relationship to other entities in the graph, for example
literature references.

The third line (`preferred_id`) informs the uniqueness of represented entities
by selecting an ontological namespace around which the definition of uniqueness
should revolve. In our example, if a protein has its own uniprot ID, it is
understood to be a unique entity. When there are multiple protein isoforms
carrying the same uniprot ID, they are understood to be aggregated to result in
only one unique entity in the graph. Decisions around uniqueness of graph
constituents sometimes require some consideration in task-specific
applications. Selection of a namespace also has effects in identifier mapping;
in our case, for protein nodes that do not carry a uniprot ID, identifier
mapping will attempt to find a uniprot ID given the other identifiers of that
node. To account for the broadest possible range of identifier systems while
also dealing with parsing of namespace prefixes and validation, we refer to the
[Bioregistry](https://bioregistry.io) project namespaces, which should be
preferred values for this field.

Finally, the fourth line (`input_label`) connects the input data stream to the
configuration; here we indicate which label to expect in the input tuple for
each class in the graph. In our case, we expect "uniprot_protein" as the label
for each protein in the input data stream; all other input entities that do not
carry this label are ignored as long as they are not in the schema
configuration.

### Creating the graph (using the BioCypher interface)
All that remains to be done now is to instantiate the BioCypher interface (as the
main means of communicating with BioCypher) and call the function to create the
graph.


```python
from biocypher import BioCypher
bc = BioCypher(
    biocypher_config_path='01_biocypher_config.yaml',
    schema_config_path='01_schema_config.yaml',
)
# Add the entities that we generated above to the graph
bc.add(entities)
```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.



```python
# Print the graph as a dictionary of pandas DataFrame(s) per node label
bc.to_df()["protein"]
```




    {'protein':   protein                                           sequence  \
     0  F7V4U2  RMFDDRFPVELRICTGSLVIINLGEFAEQHDKQDGSKPSHQPMFAT...   
     1  K2Y8U3  HWPPSGVSCGVFPECWYRWRDEQWACFGPHIKYNKDNTWSWAQWMH...   
     2  L1V6V9  QAEPKYKLAQENCRVQIKLPKIVGTCRPHWMTKTYHVLHTCVLWKS...   
     
                description taxon      id preferred_id  
     0  i f c m m q e o o s  9606  F7V4U2      uniprot  
     1  e y p g j t j y r x  9606  K2Y8U3      uniprot  
     2  a i b t l j e g n j  9606  L1V6V9      uniprot  }



## Section 2: Merging data

### Plain merge

Using the workflow described above with minor changes, we can merge data from
different input streams. If we do not want to introduce additional ontological
subcategories, we can simply add the new input stream to the existing one and
add the new label to the schema configuration (the new label being
`entrez_protein`). In this case, we would add the following to the schema
configuration:


```python
from data_generator import Protein, EntrezProtein
```


```python
print_yaml('02_schema_config.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id: uniprot
        input_label:
        - uniprot_protein
        - entrez_protein
    
    --------------



```python
# Create a list of proteins to be imported
proteins = [
    p for sublist in zip(
        [Protein() for _ in range(n_proteins)],
        [EntrezProtein() for _ in range(n_proteins)],
    ) for p in sublist
]
# Create a new BioCypher instance
bc = BioCypher(
    biocypher_config_path='02_biocypher_config.yaml',
    schema_config_path='02_schema_config.yaml',
)
# Run the import
bc.add(node_generator(proteins))
```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.



```python
bc.to_df()["protein"]
```




    {'protein':   protein                                           sequence  \
     0  K2W3K5  TVKISILFNPLPNQDMNTTTCQAESNYKAIYLYPWCSMDDVWNVEA...   
     1  186009  FHYHGGMGPFMTYQNFLHWEQMQPMKLFNEPMQFHDWYGTHVNWPG...   
     2  S6E6D1  CSVQIQIGMSQDSPDSSEGNMDCPPRNIGGYEIVCNVQGKRCYSTD...   
     3  926766  HKEAELLVKGQIQTPKCLRHNHFYAKLTIVIELNYMVDRYGKDMAR...   
     4  Z1F6R2  FMVWKDCLCIRMRHMAVPVPQYHCEYFEVILERWEVPCFSVLNRCK...   
     5  362641  PISDEQEMGSEFCGHCNTGVYQVEMHFFECEDLNPKVQPKWIFTVT...   
     
                description taxon      id preferred_id  
     0  e e v h x f t f j l  9606  K2W3K5      uniprot  
     1  b c q m l d a u u g  9606  186009      uniprot  
     2  i z t s l x v g j l  9606  S6E6D1      uniprot  
     3  t n a j d l j a t a  9606  926766      uniprot  
     4  h d m k q n r e h r  9606  Z1F6R2      uniprot  
     5  l m x k h m v g p y  9606  362641      uniprot  }



This again creates a single DataFrame, now for both protein types, but now including
both input streams (you should note both uniprot & entrez style IDs in the id column). However, we are generating our `entrez`
proteins as having entrez IDs, which could result in problems in querying.
Additionally, a strict import mode including regex pattern matching of
identifiers will fail at this point due to the difference in pattern of UniProt
vs. Entrez IDs. This issue could be resolved by mapping the Entrez IDs to
UniProt IDs, but we will instead use the opportunity to demonstrate how to
merge data from different sources into the same ontological class using *ad
hoc* subclasses.

### *Ad hoc* subclassing

In the previous section, we saw how to merge data from different sources into
the same ontological class. However, we did not resolve the issue of the
`entrez` proteins living in a different namespace than the `uniprot` proteins,
which could result in problems in querying. In proteins, it would probably be
more appropriate to solve this problem using identifier mapping, but in other
categories, e.g., pathways, this may not be possible because of a lack of
one-to-one mapping between different data sources. Thus, if we so desire, we
can merge datasets into the same ontological class by creating *ad hoc*
subclasses implicitly through BioCypher, by providing multiple preferred
identifiers. In our case, we update our schema configuration as follows:



```python
print_yaml('03_schema_config.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id:
        - uniprot
        - entrez
        input_label:
        - uniprot_protein
        - entrez_protein
    
    --------------


This will "implicitly" create two subclasses of the `protein` class, which will
inherit the entire hierarchy of the `protein` class. The two subclasses will be
named using a combination of their preferred namespace and the name of the
parent class, separated by a dot, i.e., `uniprot.protein` and `entrez.protein`.
In this manner, they can be identified as proteins regardless of their sources
by any queries for the generic `protein` class, while still carrying
information about their namespace and avoiding identifier conflicts.

<div class="alert alert-info">
The only change affected upon the code from the previous section is the
referral to the updated schema configuration file.
</div>

<div class="alert alert-success">
In the output, we now generate two separate files for the `protein` class, one
for each subclass (with names in PascalCase).
</div>

Let's create a DataFrame with the same nodes as above, but with a different schema configuration:


```python
bc = BioCypher(
    biocypher_config_path='03_biocypher_config.yaml',
    schema_config_path='03_schema_config.yaml',
)
bc.add(node_generator(proteins))
for name, df in bc.to_df().items():
    print(name)
    display(df)
```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.





    {'uniprot.protein':   uniprot.protein                                           sequence  \
     0          K2W3K5  TVKISILFNPLPNQDMNTTTCQAESNYKAIYLYPWCSMDDVWNVEA...   
     1          S6E6D1  CSVQIQIGMSQDSPDSSEGNMDCPPRNIGGYEIVCNVQGKRCYSTD...   
     2          Z1F6R2  FMVWKDCLCIRMRHMAVPVPQYHCEYFEVILERWEVPCFSVLNRCK...   
     
                description taxon      id preferred_id  
     0  e e v h x f t f j l  9606  K2W3K5      uniprot  
     1  i z t s l x v g j l  9606  S6E6D1      uniprot  
     2  h d m k q n r e h r  9606  Z1F6R2      uniprot  ,
     'entrez.protein':   entrez.protein                                           sequence  \
     0         186009  FHYHGGMGPFMTYQNFLHWEQMQPMKLFNEPMQFHDWYGTHVNWPG...   
     1         926766  HKEAELLVKGQIQTPKCLRHNHFYAKLTIVIELNYMVDRYGKDMAR...   
     2         362641  PISDEQEMGSEFCGHCNTGVYQVEMHFFECEDLNPKVQPKWIFTVT...   
     
                description taxon      id preferred_id  
     0  b c q m l d a u u g  9606  186009       entrez  
     1  t n a j d l j a t a  9606  926766       entrez  
     2  l m x k h m v g p y  9606  362641       entrez  }



Now we see two separate DataFrames, one for each subclass of the `protein` class.

## Section 3: Handling properties
While ID and label are mandatory components of our knowledge graph, properties
are optional and can include different types of information on the entities. In
source data, properties are represented in arbitrary ways, and designations
rarely overlap even for the most trivial of cases (spelling differences,
formatting, etc). Additionally, some data sources contain a large wealth of
information about entities, most of which may not be needed for the given task.
Thus, it is often desirable to filter out properties that are not needed to
save time, disk space, and memory.

<div class="alert alert-info">

Maintaining consistent properties per entity type is particularly important
when using the admin import feature of Neo4j, which requires consistency
between the header and data files. Properties that are introduced into only
some of the rows will lead to column misalignment and import failure. In
"online mode", this is not an issue.

</div>

We will take a look at how to handle property selection in BioCypher in a
way that is flexible and easy to maintain.

### Designated properties

The simplest and most straightforward way to ensure that properties are
consistent for each entity type is to designate them explicitly in the schema
configuration. This is done by adding a `properties` key to the entity type
configuration. The value of this key is another dictionary, where in the
standard case the keys are the names of the properties that the entity type
should possess, and the values give the type of the property. Possible values
are:

- `str` (or `string`),

- `int` (or `integer`, `long`),

- `float` (or `double`, `dbl`),

- `bool` (or `boolean`),

- arrays of any of these types (indicated by square brackets, e.g. `string[]`).

In the case of properties that are not present in (some of) the source data,
BioCypher will add them to the output with a default value of `None`.
Additional properties in the input that are not represented in these designated
property names will be ignored. Let's imagine that some, but not all, of our
protein nodes have a `mass` value. If we want to include the mass value on all
proteins, we can add the following to our schema configuration:


```python
print_yaml('04_schema_config.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id:
        - uniprot
        - entrez
        input_label:
        - uniprot_protein
        - entrez_protein
        properties:
            sequence: str
            description: str
            taxon: str
            mass: int
    
    --------------


This will add the `mass` property to all proteins (in addition to the three we
had before); if not encountered, the column will be empty. Implicit subclasses
will automatically inherit the property configuration; in this case, both
`uniprot.protein` and `entrez.protein` will have the `mass` property, even
though the `entrez` proteins do not have a `mass` value in the input data.

<div class="alert alert-info">
If we wanted to ignore the mass value for all properties, we could simply
remove the `mass` key from the `properties` dictionary.
</div>


```python
from data_generator import EntrezProtein, RandomPropertyProtein
```


```python
# Create a list of proteins to be imported (now with properties)
proteins = [
    p for sublist in zip(
        [RandomPropertyProtein() for _ in range(n_proteins)],
        [EntrezProtein() for _ in range(n_proteins)],
    ) for p in sublist
]
# New instance, populated, and to DataFrame
bc = BioCypher(
    biocypher_config_path='04_biocypher_config.yaml',
    schema_config_path='04_schema_config.yaml',
)
bc.add(node_generator(proteins))
for name, df in bc.to_df().items():
    print(name)
    display(df)
```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.





    {'uniprot.protein':   uniprot.protein                                           sequence  \
     0          S1Z9L5  RHLRGDVMQEDHHTSSERMVYNVLPQDYKVVSCEYWNTQVTALWVI...   
     1          W9J5F1  IPFSQSAWAQQRIGPKGTKAHGVTQPAPMDIKNLCNLTDLTLILDF...   
     2          T1J3U0  WFGCCHKQYVSHVIDRQDPQSPSDNPSLVSQLQFFMWGIQIQNGEI...   
     
                description taxon  mass      id preferred_id  
     0  u x e o k m a i o s  3899  None  S1Z9L5      uniprot  
     1  i x k c r b p d d p  8873  None  W9J5F1      uniprot  
     2  m a w r r u x c w o  1966  9364  T1J3U0      uniprot  ,
     'entrez.protein':   entrez.protein                                           sequence  \
     0         405878  RMTDGFEWQLDFHAFIWCNQAAWQLPLEVHISQGNGGWRMGLYGNM...   
     1         154167  CGMNYDNGYFSVAYQSYDLWYHQQLKTRGVKPAEKDSDKDLGIDVI...   
     2         234189  GQWQECIQGFTPQQMCVDCCAETKLANKSYYHSWMTWRLSGLCFNM...   
     
                description taxon  mass      id preferred_id  
     0  y c s v s n e c h o  9606  None  405878       entrez  
     1  i k n c e n r n c d  9606  None  154167       entrez  
     2  o v w y g h y e v y  9606  None  234189       entrez  }



### Inheriting properties
Sometimes, explicit designation of properties requires a lot of maintenance
work, particularly for classes with many properties. In these cases, it may be
more convenient to inherit properties from a parent class. This is done by
adding a `properties` key to a suitable parent class configuration, and then
defining inheritance via the `is_a` key in the child class configuration and
setting the `inherit_properties` key to `true`.

Let's say we have an additional `protein isoform` class, which can reasonably
inherit from `protein` and should carry the same properties as the parent. We
can add the following to our schema configuration:


```python
from data_generator import RandomPropertyProteinIsoform
```


```python
print_yaml('05_schema_config.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id:
        - uniprot
        - entrez
        input_label:
        - uniprot_protein
        - entrez_protein
        properties:
            sequence: str
            description: str
            taxon: str
            mass: int
    protein isoform:
        is_a: protein
        inherit_properties: true
        represented_as: node
        preferred_id: uniprot
        input_label: uniprot_isoform
    
    --------------


This allows maintenance of property lists for many classes at once. If the child
class has properties already, they will be kept (if they are not present in the
parent class) or replaced by the parent class properties (if they are present).

Again, apart from adding the protein isoforms to the input stream, the code
for this example is identical to the previous one except for the reference to
the updated schema configuration.

We now create three separate DataFrames, all of which are children of the
`protein` class; two implicit children (`uniprot.protein` and `entrez.protein`)
and one explicit child (`protein isoform`).



```python
# create a list of proteins to be imported
proteins = [
    p for sublist in zip(
        [RandomPropertyProtein() for _ in range(n_proteins)],
        [RandomPropertyProteinIsoform() for _ in range(n_proteins)],
        [EntrezProtein() for _ in range(n_proteins)],
    ) for p in sublist
]

# Create BioCypher driver
bc = BioCypher(
    biocypher_config_path='05_biocypher_config.yaml',
    schema_config_path='05_schema_config.yaml',
)
# Run the import
bc.add(node_generator(proteins))

for name, df in bc.to_df().items():
    print(name)
    display(df)
```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.


    uniprot.protein
      uniprot.protein                                           sequence  \
    0          A9L6G4  SWIVVGQPDSHNKRLVNYHWMRCEHPLRCWRPIYVVRVSFQSQCEQ...   
    1          E4N2H2  PGVMILDNMQHKCSKELSTRQIITNHWICNSAPISWSSGMDRSCLD...   
    2          V4F1T1  DQCHNLCPGSSFQCPENAFGNDWIDHMPQETGLMQYDDPQSGMWFT...   
    
               description taxon  mass      id preferred_id  
    0  m o k j a f w v w r  4220  None  A9L6G4      uniprot  
    1  n v i r s f m f d w  6339  6481  E4N2H2      uniprot  
    2  w e v v a b o b b u  9176  6510  V4F1T1      uniprot  
    protein isoform
      protein isoform                                           sequence  \
    0          F0N9A4  QDVVLVEGCGDEGWIHMPEKRPGQAYKWCERFRPIPDFTNSIKIAY...   
    1          B1W6O2  SQKHFRRWWTNDCFGQELMSIYYNVKFWDNLIEMTGGPASRVCLGQ...   
    2          G6V5R9  ASAITPFSYEKPHTVTLDATEVFPKMQDAQAIEREIHFSKSTLVYG...   
    
               description taxon  mass      id preferred_id  
    0  r f e a v a a g w r  8061  None  F0N9A4      uniprot  
    1  a c a v v k v k c w  6786  None  B1W6O2      uniprot  
    2  c k g d a l f r t v  6868  1323  G6V5R9      uniprot  
    entrez.protein
      entrez.protein                                           sequence  \
    0          52329  DYRSMAPTFILMKIYPACDAITKRRWSVATVKDGEFIWWSAVKIFP...   
    1         581107  LLVFNMGQLAVAGYGNTMVSAMMCFCCDVKARMGMSWLPKITTMQW...   
    2         270569  MVCSHHELAVAFQTMCPIQGDAATAKANAHRTTDKQNWMVVKWFRT...   
    
               description taxon  mass      id preferred_id  
    0  q k r b h g t q x x  9606  None   52329       entrez  
    1  h f g z j r b g m w  9606  None  581107       entrez  
    2  s b p v f u t y g v  9606  None  270569       entrez  


## Section 4: Handling relationships

Naturally, we do not only want nodes in our knowledge graph, but also edges. In
BioCypher, the configuration of relationships is very similar to that of nodes,
with some key differences. First the similarities: the top-level class
configuration of edges is the same; class names refer to ontological classes or
are an extension thereof. Similarly, the `is_a` key is used to define
inheritance, and the `inherit_properties` key is used to inherit properties from
a parent class. Relationships also possess a `preferred_id` key, an
`input_label` key, and a `properties` key, which work in the same way as for
nodes.

Relationships also have a `represented_as` key, which in this case can be
either `node` or `edge`. The `node` option is used to "reify" the relationship
in order to be able to connect it to other nodes in the graph. In addition to
the configuration of nodes, relationships also have fields for the `source` and
`target` node types, which refer to the ontological classes of the respective
nodes, and are currently optional.

To add protein-protein interactions to our graph, we can modify the schema configuration above to the following:


```python
print_yaml('06_schema_config_pandas.yaml')
```

    --------------
    protein:
        represented_as: node
        preferred_id:
        - uniprot
        - entrez
        input_label:
        - uniprot_protein
        - entrez_protein
        properties:
            sequence: str
            description: str
            taxon: str
            mass: int
    protein isoform:
        is_a: protein
        inherit_properties: true
        represented_as: node
        preferred_id: uniprot
        input_label: uniprot_isoform
    protein protein interaction:
        is_a: pairwise molecular interaction
        represented_as: edge
        preferred_id: intact
        input_label: interacts_with
        properties:
            method: str
            source: str
    
    --------------


Now that we have added `protein protein interaction` as an edge, we have to simulate some interactions:


```python
from data_generator import InteractionGenerator

# Simulate edges for proteins we defined above
ppi = InteractionGenerator(
    interactors=[p.get_id() for p in proteins],
    interaction_probability=0.05,
).generate_interactions()
```


```python
# naturally interactions/edges contain information about the interacting source and target nodes
# let's look at the first one in the list
interaction = ppi[0]
f"{interaction.get_source_id()} {interaction.label} {interaction.get_target_id()}"
```




    'A9L6G4 interacts_with V4F1T1'




```python
# similarly to nodes, it also has a dictionary of properties
interaction.get_properties()
```




    {'source': 'signor', 'method': 'u z c x m d c u g s'}



As with nodes, we add first createa a new BioCypher instance, and then populate it with nodes as well as edges:


```python
bc = BioCypher(
    biocypher_config_path='06_biocypher_config.yaml',
    schema_config_path='06_schema_config_pandas.yaml',
)
```


```python
# Extract id, source, target, label, and property dictionary
def edge_generator(ppi):
    for interaction in ppi:
        yield (
            interaction.get_id(),
            interaction.get_source_id(),
            interaction.get_target_id(),
            interaction.get_label(),
            interaction.get_properties(),
        )

bc.add(node_generator(proteins))
bc.add(edge_generator(ppi))

```

    INFO -- Loading ontologies...
    INFO -- Instantiating OntologyAdapter class for https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl.


Let's look at the interaction DataFrame:


```python
bc.to_df()["protein protein interaction"]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>protein protein interaction</th>
      <th>_from</th>
      <th>_to</th>
      <th>source</th>
      <th>method</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>intact703256</td>
      <td>A9L6G4</td>
      <td>V4F1T1</td>
      <td>signor</td>
      <td>u z c x m d c u g s</td>
    </tr>
    <tr>
      <th>1</th>
      <td>None</td>
      <td>E4N2H2</td>
      <td>F0N9A4</td>
      <td>intact</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



Finally, it is worth noting that BioCypher relies on ontologies, which are machine readable representations of domains of knowledge that we use to ground the contents of our knowledge graphs. While details about ontologies are out of scope for this tutorial, and are described in detail in the [BioCypher documentation](https://biocypher.org/tutorial-ontology.html), we can still have a glimpse at the ontology that we used implicitly in this tutorial:


```python
bc.show_ontology_structure()
```

    Showing ontology structure based on https://github.com/biolink/biolink-model/raw/v3.2.1/biolink-model.owl.ttl
    entity
    ├── association
    │   └── gene to gene association
    │       └── pairwise gene to gene interaction
    │           └── pairwise molecular interaction
    │               └── protein protein interaction
    └── named thing
        └── biological entity
            └── polypeptide
                └── protein
                    ├── entrez.protein
                    ├── protein isoform
                    └── uniprot.protein





    <treelib.tree.Tree at 0x7f7327b3a880>


