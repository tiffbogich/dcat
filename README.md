dcat
====

Archive and make discoverable data and links with
[schema.org](http://schema.org) metadata.

[![NPM](https://nodei.co/npm/dcat.png)](https://nodei.co/npm/dcat/)

Usage (CLI)
===========

## tl;dr

    dcat --help


## Registering an User (```adduser```)

Run

    dcat adduser

and follow the prompting wizard.

## Publishing (```publish```)

### Simple document

```dcat``` allows the publication of [JSON-LD](http://www.w3.org/TR/json-ld)
documents using [dcat.io](https://dcat.io) context. This context
extends [schema.org](http://schema.org) with terms relevant to do I/O
and preserve data integrity (like ```filepath``` and ```Checksum```).

At the minimum, a document has to contain

- a context (```@context```) set to https://dcat.io,
- an id (```@id```) to uniquely identify things published on
  dcat.io with URLs. All relative URLs will be resolved with a base
  (defined in the [context](https://dcat.io) (```@base```)) of
  ```https://dcat.io```

e.g.

    {
      "@context": "https://dcat.io",
      "@id": "mydoc"
    }

To publish this document (```mydoc```), create a blank file named ```JSONLD``` and in the directory containing ```JSONLD``` run:

    dcat publish

After publication the document will be available at ```https://dcat.io/mydoc```.

Documents can contain any properties from
[schema.org](http://schema.org) or from any other ontologies as long
as the associated ```@context``` are provided.

### Versioning

If a [```version```](http://schema.org/version) property is specified
in the document, the document will be versioned, that is, each update
will require a new version value in order to be published (this prevents
existing versions from being overwritten).

When appropriate version number SHOULD follow
[semantic versioning](http://semver.org/)

e.g.

    {
      "@context": "https://dcat.io",
      "@id": "mydoc",
      "version": "0.0.1"
    }

After publication this document will be available at
```https://dcat.io/mydoc?version=0.0.1``` whereas the latest version
will always be available at ```https://dcat.io/mydoc```.

In case the document is versioned following
[Semantic Versioning](http://semver.org/), a range (e.g. ```<0.0.1```)
can be specified as ```version``` (e.g. ```https://dcat.io/mydoc?version=<0.0.1```)

### Nodes

Document can be arbitrarily complex (having multiple nodes) and
sometimes, it makes sense to assign a URL to a node so that
it can be referenced. This is achieved by setting ```@id``` properties
to the desired nodes 

e.g.


    {
      "@context": "https://dcat.io",
      "@id": "mydoc",
      "version": "0.0.1",
      "hasPart": {
        "@id": "mydoc/data",
        "@type": "Dataset",
        "description": "a dataset part of the document"
      }
    }


The whole document can be retrieved at ```https://dcat.io/mydoc```
whereas the node (part) can be retrieved at ```https://dcat.io/mydoc/data```

Note: nodes can be any valid URLs _but_ they have to be namespaced
within the top level ```@id``` (for a document of ```""@id":
"mydoc""```, ```"@id": "mydoc/arbitrarily/long/pathname"``` will be
valid whereas ```"@id": "part"``` won't).

### Adding metadata to existing URLs


```dcat``` can be used to add _machine readable_ metadata to any
resources already published on the web.
For instance running:

    dcat init https://github.com/standard-analytics/dcat.git

we get a basic machine readable document:

    {
      "@context": "https://dcat.io",
      "@id": "mydoc",
      "@type": "Code",
      "codeRepository": "https://github.com/standard-analytics/dcat",
      "encoding":  {
        "@type": "MediaObject",
         "contentUrl": "https://api.github.com/repos/standard-analytics/dcat/tarball/master",
         "encodingFormat": "application/x-gzip",
         "contentSize": 690980
       }
    }

This document should be extended with more properties (from
[schema.org](http://schema.org) such as
[author](http://schema.org/author),
[contributor](http://schema.org/contributor),
[about](http://schema.org/about),
[programmingLanguage](http://schema.org/programmingLanguage),
[runtime](http://schema.org/runtime)..., or from any other web ontologies, taking care to add contexts in this case) to improve the
discoverability and reusability of the resource.

Note, in addition to absolute URLs, ```dcat``` supports
[CURIE](http://www.w3.org/TR/curie/) for the prefixes defined in the
dcat.io ```@context```. Using a CURIE, the previous is simplified to:

    dcat init github:standard-analytics/dcat.git


### Files (raw data)

For all the subclasses of
[schema.org/CreativeWork](http://schema.org/CreativeWork) (e.g
[Dataset](http://schema.org/Dataset), [Code](http://schema.org/Code),
[SoftwareApplication](http://schema.org/SoftwareApplication),
[Article](http://schema.org/Article), [Book](http://schema.org/Book),
[ImageObject](http://schema.org/ImageObject),
[VideoObject](http://schema.org/VideoObject),
[AudioObject](http://schema.org/AudioObject), ...) ```dcat``` allows
the publication of raw data from files (including datasets, binaries, images, media, and more...)
along with documents.

For instance if you have an a
[PDF](http://en.wikipedia.org/wiki/Portable_Document_Format) of a
[MedicalScholarlyArticle](http://schema.org/MedicalScholarlyArticle)
and an associated [Dataset](http://schema.org/Dataset) in
[CSV](http://tools.ietf.org/html/rfc4180) you can run:

    dcat init --main article.pdf::MedicalScholarlyArticle --part data.csv

Note: ```::MedicalScholarlyArticle``` associates a type
(```@type```) with the resource (```article.pdf```).

This will generate a machine readable document (JSONLD) that you can
edit to provide additional metadata.

    {
      "@context": "https://dcat.io",
      "@id": "mydoc",
      "@type": "MedicalScholarlyArticle",
      "encoding": {
        "@type": "MediaObject",
        "filePath": "article.pdf"
      },
      "hasPart": {
        "@type": "Dataset",
        "distribution": {
          "@type": "DataDownload",
          "filePath": "data.csv"
        }
      }
    }

After publication (```dcat publish```) the document will acquire
additional URL properties that can be dereferenced to retrieved the
original raw data:


    {
      "@context": "https://dcat.io",
      "@id": "mydoc",
      "@type": "MedicalScholarlyArticle",
      "encoding": {
        "@type": "MediaObject",
        "filePath": "article.pdf",
        "contentUrl": "http://example.com/article.pdf" //generated URL
      },
      "hasPart": {
        "@type": "Dataset",
        "distribution": {
          "@type": "DataDownload",
          "filePath": "data.csv",
          "contentUrl": "http://example.com/data.csv" //generated URL
        }
      }
    }


Note: ```dcat init``` supports [globbing](https://github.com/isaacs/node-glob) so you can run commands like:

    dcat init --main article.pdf --part *.csv

or repeat ```--part``` (or the shorter ```-p```) if you need more complex matching e.g.

    dcat init --m article.pdf -p *.csv -p *.jpg


#### Directories

Directories are published as tarballs. For instance, running


    dcat init -m src::Code --id cproject


where ```src``` is a directory of source files


```
src
├── lib.h
└── main.c
```

will generate:

    {
      "@context": "https://dcat.io",
      "@id": "cproject",
      "@type": "Code",
      "programmingLanguage": { "name": "c" },
      "encoding": {
        "@type": "MediaObject",
        "encodingFormat": "application/x-gtar",
        "hasPart": [
          { "@type": "MediaObject", "filePath": "src/lib.h" },
          { "@type": "MediaObject", "filePath": "src/main.c" }
        ]
      }
    }


After publication, the MediaObject will have a
[```contentUrl```](http://www.schema.org/contentUrl) property
indicating where the tarball can be retrieved.


## Unpublishing (```unpublish```)

To delete a specific version of a document of ```"@id": "mydoc"``` run:

    dcat unpublish ldr:mydoc?version=0.1.1

```ldr``` is the prefix used for ```https://dcat.io``` (defined in the
dcat.io ```@context```).

To delete all versions of a document of ```"@id": "mydoc"``` run:

    dcat unpublish ldr:mydoc


## Retrieving documents and raw data (```search```, ```show```, ```clone```)

### Search

Document containing [keywords](http://schema.org/keywords),
[name](http://schema.org/name) or
[description](http://schema.org/description) properties can be
searched by keyword with ```dcat search``` followed by a list of
keywords.

For more powerful search, all data published on
[dcat.io](https://dcat.io) are valid
[linked data fragments](http://linkeddatafragments.org/) and can be
queried using [SPARQL](http://www.w3.org/TR/rdf-sparql-query/).

### Show (expanded, compacted, flattened, normalized )

```dcat show``` followed by a [CURIE](http://www.w3.org/TR/curie/)
displays the latest
JSON-LD document corresponding to the CURIE on
[stdout](http://en.wikipedia.org/wiki/Standard_streams).

Different options (```-e, --expand```, ```-f, --flatten```, ```-c,
--compact```, ```-n, --normalize```) provide alternative
representations of the document. For instance,

```dcat show ldr:mydoc?version=<2.1.0 --normalize```

will serialize the latest version smaller than 2.1.0 of the document
of ```"@id": "mydoc"``` to [N-Quads](http://www.w3.org/TR/n-quads/)
([RDF](http://www.w3.org/2007/02/turtle/primer/)).

### Clone

```dcat clone``` followed by a [CURIE](http://www.w3.org/TR/curie/) downloads the raw data associated with a document and stores them along with the document on
disk at the paths specified by the ```filepath``` properties.


## Listing / Adding / Removing maintainers (```maintainer```)

Only maintainers of a document can publish or remove versions of a
document. Maintainers of a document can be listed with:

    dcat maintainer ls <CURIE>

Maintainers can give users maintainer rights by running:

    dcat maintainer add <user CURIE> <doc CURIE>

Note: all user of [dcat.io](https://dcat.io) have a [CURIE](http://www.w3.org/TR/curie/) of ```ldr:users/{username}```

Maintainers can remove maintainer rights by running:

    dcat maintainer rm <user CURIE> <doc CURIE>


API
===

```dcat``` can also be used programmatically.

    var Dcat = require('dcat');
    var dcat = new Dcat();

    var doc = {
      '@context': 'https://dcat.io,
      '@id': 'test',
      name: 'hello world'
    };

    dcat.publish(doc, function(err, cdoc){
      console.log(err, cdoc); //cdoc is compacted
    });

See ```test/test.js``` for more examples.

History
=======

[```package.json```](http://wiki.commonjs.org/wiki/Packages/1.1) -> [```datapackage.json```](http://dataprotocols.org/data-packages/) -> ```package.jsonld``` -> [```JSON-LD```](http://json-ld.org/) + [schema.org](http://schema.org) + [hydra](http://www.hydra-cg.com/) + [linked data fragment](http://www.hydra-cg.com/).


Registry
========

By default, ```dcat``` uses [dcat.io](http://dcat.io), a
[linked data registry](https://github.com/standard-analytics/linked-data-registry)
hosted on [cloudant](https://standardanalytics.cloudant.com).

Tests
=====

You need a local instance of the [linked data registry](https://github.com/standard-analytics/linked-data-registry) running on your machine on port 3000. Then, run:

    npm test


License
=======

Apache-2.0.
