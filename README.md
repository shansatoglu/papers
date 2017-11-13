![travis](https://travis-ci.org/perrette/myref.svg?branch=master)
[![Coverage Status](https://coveralls.io/repos/github/perrette/myref/badge.svg?branch=master)](https://coveralls.io/github/perrette/myref?branch=master)
# myref

Command-line tool to manage bibliography (pdfs + bibtex)


Motivation
----------
This project is an attempt to create a light-weight, 
command-line bibliography managenent tool. Aims:

- maintain a PDF library (with appropriate naming)
- maintain one or several bibtex-compatible collections, linked to PDFs
- some PDF-parsing capability (especially to extract DOI)
- fetch PDF metadata from the internet (i.e. [crossref](https://github.com/CrossRef/rest-api-doc)), preferably based on DOI


Why not JabRef, Zotero or Mendeley (or...) ?
--------------------------------------------
- JabRef (2.10) is nice, light-weight, but is not so good at managing PDFs.
- Zotero (5.0) features excellent PDF import capability, but it needs to be manually one by one and is a little slow .
- Mendeley (1.17) is perfect at automatically extracting metadata from downloaded PDF and managing your PDF library, 
but it is not open source, and many issues remain (own experience, Ubuntu 14.04, Desktop Version 1.17):
    - very unstable
    - PDF automatic naming is too verbose, and sometimes the behaviour is unexpected (some PDFs remain in on obscure Downloaded folder, instead of in the main collection)
    - somewhat heavy (it offers functions of online syncing, etc)
    - poor seach capability (related to the point above)

Above-mentioned issues will with no doubt be improved in future releases, but they are a starting point for this project.
Anyway, a command-line tool is per se a good idea for faster development, 
as noted [here](https://forums.zotero.org/discussion/43386/zotero-cli-version), 
but so far I could only find zotero clients for their online API 
(like [pyzotero](https://github.com/urschrei/pyzotero) or [zotero-cli](https://github.com/jbaiter/zotero-cli)).
Please contact me if you know another interesting project.


Internals
---------
For now (very much beta version), the project:
- manages one `bibtex` collection, maintained sorted according to entry keys
- [bibtexparser](https://bibtexparser.readthedocs.io/en/v0.6.2) is used to parse bibtexentries and libraries
- each entry (and associated keys) is obtained from [crossref API](https://github.com/CrossRef/rest-api-doc/issues/115#issuecomment-221821473) (note: the feature that allows to fetch a bibtex entry from DOI is undocumented...). So far it seems that the keys are unique, until seen otherwise...
- DOI is extracted from PDFs with regular expression search within the first two pages.
- TODO: add other means to retrieve metadata, such as `ISBN`. Get inspired from [Zotero](https://forums.zotero.org/discussion/57418/retrieve-pdfs-metadata-wrong-metadata-source).

Dependencies
------------
- python 2 or 3
- `pdftotext` (third-party): convert PDF to text
    - Tested with v0.41
    - Natively installed on Ubuntu 14.04 (?). Part of poppler-utils.

- [bibtexparser](https://bibtexparser.readthedocs.io/en/v0.6.2)
    - pip install bibtexparser

- [crossrefapi](https://github.com/fabiobatalha/crossrefapi) : make polite requests to crossref API
    - pip install crossrefapi

- [scholarly](https://github.com/OrganicIrradiation/scholarly) : interface for google scholar
    - pip install scholarly

- [fuzzywuzzy](https://github.com/seatgeek/fuzzywuzzy) : calculate score to sort crossref requests
    - pip install fuzzywuzzy


Install
-------
- clone this project
- python setup.py install


Getting started
---------------
This tool's interface is built like `git`, with main command `myref` and a range of subcommands.

Start with PDF of your choice (modern enough to have a DOI, e.g. anything from the Copernicus publications). 
For the sake of the example, one of my owns: https://www.earth-syst-dynam.net/4/11/2013/esd-4-11-2013.pdf

- extract pdf metadata (doi-based if available, otherwise crossref, or google scholar if so specified)

        $> myref extract esd-4-11-2013.pdf
        @article{Perrette_2013,
            doi = {10.5194/esd-4-11-2013},
            url = {https://doi.org/10.5194%2Fesd-4-11-2013},
            year = 2013,
            month = {jan},
            publisher = {Copernicus {GmbH}},
            volume = {4},
            number = {1},   
            pages = {11--29},
            author = {M. Perrette and F. Landerer and R. Riva and K. Frieler and M. Meinshausen},
            title = {A scaling approach to project regional sea level rise and its uncertainties},
            journal = {Earth System Dynamics}
        }

- add pdf to `myref.bib`  library, and rename a copy of it in a files directory `files`.

        $> myref add --rename --copy --bibtex myref.bib --filesdir files esd-4-11-2013.pdf --info
    	INFO:root:found doi:10.5194/esd-4-11-2013
    	INFO:root:new entry: perrette_2013
    	INFO:root:create directory: files/2013
    	INFO:root:mv /home/perrette/playground/myref/esd-4-11-2013.pdf files/2013/Perrette_2013.pdf
    	INFO:root:renamed file(s): 1

(the `--info` argument asks for the above output information to be printed out to the terminal)

In the common case where the bibtex (`--bibtex`) and files directory  (`--filesdir`) do not change, 
it is convenient to *install* `myref`. 
Install comes with the option to git-track any change to the bibtex file (`--git`) options.

- setup git-tracked library (optional)

        $> myref install --bibtex myref.bib --filesdir files --git --gitdir ./
        myref configuration
        * configuration file: /home/perrette/.config/myrefconfig.json
        * cache directory:    /home/perrette/.cache/myref
        * git-tracked:        True
        * git directory :     ./
        * files directory:    files (1 files, 5.8 MB)
        * bibtex:            myref.bib (1 entries)

Note the existing bibtex file was detected but untouched.
The configuration file is global (unless `--local` is specified), so from now on, any `myref` 
command will be know about these settings. Type `myref status -v` to check your
configuration.
You also notice that crossref requests are saved in the cache directory. 
This happens regardless of whether `myref` is installed or not.
From now on, no needs to specify bibtex file or files directory.

- list entries (and edit etc...)

        $> myref list -l
        Perrette2013: A scaling approach to project regional sea level rise and it... (doi:10.5194/esd-4-11-2013, file:1)

`myref list` is a useful command, inspired from unix's `find` and `grep`. 
It lets you search in your bibtex in a typical manner (including a number of special flags such as `--duplicates`, `--review-required`, `--broken-file`...), 
then output the result in a number of formats (one-liner, raw bibtex, keys-only, selected fields) or let you perform actions on it (currently `--edit`, `--delete`).
For instance, it is possible to manually merge the duplicates with:

        $> myref list --duplicates --edit


- other commands: 

    - `myref status ...` 
    - `myref check ...` 
    - `myref filecheck ...` 
    - `myref undo ...` 
    - `myref git ...` 

Consult inline help for more detailed documentation!


Current features
----------------
- parse PDF to extract DOI
- fetch bibtex entry from DOI (using crossref API)
- fetch bibtex entry by fulltext search (using crossref API or google scholar)
- create and maintain bibtex file
- add entry as PDF (`myref add ...`)
- add entry as bibtex (`myref add ...`)
- scan directory for PDFs (`myref add ...`)
- rename PDFs according to bibtex key and year (`myref filecheck --rename [--copy]`)
- some support for attachment
- merging (`myref check --duplicates ...`)
- undo command (`myref undo`)
- configuration file with default bibtex and files directory (`myref install --bibtex BIB --filesdir DIR ...`)
- integration with git (`myref install --git --gitdir DIR` and e.g. `myref git ...` to setup a remote, push...)
- display / search / list entries : format as bibtex or key or whatever (`myref list ... [-k | -l]`)
- list + edit or remove entry by key or else  (`myref list ... [--edit, --delete]`)
- fix broken PDF links (`myref filecheck ...`):
    - remove duplicate file names (always) or file copies (`--hash-check`)
    - remove missing link (`--delete-missing`)
    - fix files name after a Mendeley export (`--fix-mendeley`):
        - leading '/' missing is added
        - latex characters, e.g. `{\_}` or `{\'{e}}` replaced with unicode


Planned features
----------------
- move library location (i.e. both on disk and in bibtex's `file` entry)
- support collections (distinct bibtex entries, same files directory)


Tests
-----
Test coverage very probably lagging being features' addition, but in progress.
