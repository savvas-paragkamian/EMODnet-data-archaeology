# Workflow for legacy literature annotation EMODnet

Legacy literature contains valuable information about biodiversity. Dedicated workflows are needed in order to extract this information and transform it in structured data format. This is process is a multiple step process requiring many tools and interdisciplinary knowledge. In 2015, a [workshop](httpse//riojournal.com/articles.php?journal_name=rio&id=10445) was help in [HCMR](https://www.hcmr.gr/en/) to standardise this process within the framework of [EMODnet biology](https://www.emodnet-biology.eu). 

A new, upgraded report (part of EMODnet Phase III, available [here](https://www.emodnet-biology.eu/sites/emodnet-biology.eu/files/public/documents/EMODnet_Biology_III/Deliverables/D3.7.pdf)) was released on 07/12/2020 that focuses on the comparison of different tools and interfaces in order to automate and assist the curation process. Specifically, tools in terms of OCR and text mining technologies were tested and reviewed with the aim to design a workflow that can accommodate the need for automation and acceleration in digitising historical datasets and extracting their information. Two types of curation workflows were described as shown in Figure 1; one that relies on web applications and the other that combines programming libraries and packages. The latter is scalable, customisable and replicable but requires programming skills whereas the former is easy to implement through Graphical User Interfaces (GUI) at the expense of the previous advantages.

![Figure 1. The proposed workflows with the available tools. On the left, the GUI web applications are dispayed and on the right the programming libraries and command line tools.](gui-cli-workflows.png)

This repository is supplementary to this report for the programming / Command Line Interface workflow.

## Prerequisites

All the following code was tested on a mac running macOS Catalina 10.15.7.

### System Tools

* GNU bash, version 3.2.57
* R version 4.0.3
* perl v5.32.0
* awk version 20070501

### Workflow Tools



## Scan to pdf

Scanning expedition reports, research articles and books has been well underway.

## Single page PNG

```
convert -density 400 legacy-literature.pdf -quality 100 tool-testing-template.png
```

## OCR
To scan tables use [PyTesseract](https://fazlurnu.com/2020/06/23/text-extraction-from-a-table-image-using-pytesseract-and-opencv/)

```
for f in *.png; do tesseract -l eng $f ${f%".png"}; done
```

Then we can combine all the pages into one txt document

```
cat *.txt >> all.txt
```

## EXTRACT species and environments and tissues

```
./getEntities_EXTRACT_api.pl tool-testing-template.txt > tool-template-extract.tsv
```

EXTRACT returns the NCBI ids. We can later transform them to names by :

1. download the ttps://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz

```
wget https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz
```
2. use the node.dmp (we need the tax id and rank id columns) and names.dmp (we need the tax id and names in text)
3. change the delimiter from \t|\t to \t

```
more nodes.dmp | sed 's/:ctrl-v-tab:\|//g' > nodes_tab.tsv

more names.dmp | sed 's/:ctrl-v-tab:\|//g' > names_tab.tsv
```
4. merge the node.dmp and names.dmp based on the first column

```
awk -F'\t' 'FNR==NR{a[$1]=$3;next} ($1 in a) {print $1,a[$1],$2}' nodes_tab.tsv names_tab.tsv > ncbi_nodes_names.tsv
```
5. Remove the NCBI prefix of EXTRACT 
6. Merge the files


## gnfinder


```
gnfinder find tool-testing-template.txt > tool-testing-gnfinder.json
```

This command line tool returns a json file that has 2 arrays, metadata and names.

To extract the names

```
more tool-template-gnfinder.json | jq '.names[] | {name: .name}'

more tool-template-gnfinder.json | jq '.names[] | {name: .name} | [.name] | @tsv' | sed 's/"//g' > tool-template-gnfinder-species.tsv
```
#Code for worms

