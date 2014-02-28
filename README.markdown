FileSystem River for Elasticsearch
==================================

Welcome to the FS River Plugin for [Elasticsearch](http://www.elasticsearch.org/)

This river plugin helps to index documents from your local file system and using SSH.

**WARNING**: If you use this river in a multinode mode on different servers without SSH, you need to ensure that the river
can access files on the same mounting point. If not, when a node stop, the other node will _think_ that your
local dir is empty and will **erase** all your docs.

**WARNING**: starting from 0.0.3, you need to have the [Attachment Plugin](https://github.com/elasticsearch/elasticsearch-mapper-attachments). It's not included anymore
in the distribution.

**WARNING**: starting from 0.4.0, you don't need anymore the Attachment Plugin as we use now directly
[Tika](http://tika.apache.org/), see [#38](https://github.com/dadoonet/fsriver/issues/38).

Please read documentation relative to the version you are using:

* [0.4.0](https://github.com/dadoonet/fsriver/blob/fsriver-0.4.0/README.markdown#filesystem-river-for-elasticsearch)
* [0.3.0](https://github.com/dadoonet/fsriver/blob/fsriver-0.3.0/README.markdown#filesystem-river-for-elasticsearch)
* [0.2.0](https://github.com/dadoonet/fsriver/blob/fsriver-0.2.0/README.markdown#filesystem-river-for-elasticsearch)
* [0.1.0](https://github.com/dadoonet/fsriver/blob/fsriver-0.1.0/README.markdown#filesystem-river-for-elasticsearch)
* [0.0.3](https://github.com/dadoonet/fsriver/blob/fsriver-0.0.3/README.markdown#filesystem-river-for-elasticsearch)
* [0.0.2](https://github.com/dadoonet/fsriver/blob/fsriver-0.0.2/README.markdown#filesystem-river-for-elasticsearch)
* [0.0.1](https://github.com/dadoonet/fsriver/blob/fsriver-0.0.1/README.markdown#filesystem-river-for-elasticsearch)

Versions
--------

|      FS River Plugin    | elasticsearch | Attachment Plugin | Tika | Release date |
|-------------------------|:-------------:|:-----------------:|:----:|:------------:|
| 0.6.0-SNAPSHOT (master) |    1.0.1      |      Not used     |  1.4 |              |
| 0.5.0-SNAPSHOT (master) |    0.90.7     |      Not used     |  1.4 |              |
| 0.4.0                   |    0.90.7     |      Not used     |  1.4 |  22/12/2013  |
| 0.3.0                   |    0.90.3     |       1.8.0       |      |  09/08/2013  |
| 0.2.0                   |    0.90.0     |       1.7.0       |      |  30/04/2013  |
| 0.1.0                   | 0.90.0.Beta1  |       1.6.0       |      |  15/03/2013  |
| 0.0.3                   |    0.20.4     |       1.6.0       |      |  12/02/2013  |
| 0.0.2                   |    0.19.8     |       1.4.0       |      |  16/07/2012  |
| 0.0.1                   |    0.19.4     |       1.4.0       |      |  19/06/2012  |

Build Status
------------

Thanks to cloudbees for the [build status](https://buildhive.cloudbees.com/job/dadoonet/job/fsriver/) : 
![build status](https://buildhive.cloudbees.com/job/dadoonet/job/fsriver/badge/icon "Build status")

[![Test trends](https://buildhive.cloudbees.com/job/dadoonet/job/fsriver/test/trend)](https://buildhive.cloudbees.com/job/dadoonet/job/fsriver/)


Getting Started
===============

Installation
------------

Just type :

```sh
bin/plugin -install fr.pilato.elasticsearch.river/fsriver/0.4.0
```

This will do the job...

```
-> Installing fr.pilato.elasticsearch.river/fsriver/0.4.0...
Trying http://download.elasticsearch.org/fr.pilato.elasticsearch.river/fsriver/fsriver-0.4.0.zip...
Trying http://search.maven.org/remotecontent?filepath=fr/pilato/elasticsearch/river/fsriver/0.4.0/fsriver-0.4.0.zip...
Trying https://oss.sonatype.org/service/local/repositories/releases/content/fr/pilato/elasticsearch/river/fsriver/0.4.0/fsriver-0.4.0.zip...
Downloading ......DONE
Installed fsriver
```

Creating a Local FS river
-------------------------

We create first an index to store our *documents* :

```sh
curl -XPUT 'localhost:9200/mydocs/' -d '{}'
```

We create the river with the following properties :

* FS URL: `/tmp` or `c:\\tmp` if you use Microsoft Windows OS
* Update Rate: every 15 minutes (15 * 60 * 1000 = 900000 ms)
* Get only docs like `*.doc` and `*.pdf`
* Don't index `resume*`


```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 900000,
	"includes": "*.doc,*.pdf",
	"excludes": "resume"
  }
}'
```

Adding another local FS river
-----------------------------

We add another river with the following properties :

* FS URL: `/tmp2`
* Update Rate: every hour (60 * 60 * 1000 = 3600000 ms)
* Get only docs like `*.doc`, `*.xls` and `*.pdf`

By the way, we define to index in the same index/type as the previous one:

* index: `docs`
* type: `doc`

```sh
curl -XPUT 'localhost:9200/_river/mynewriver/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp2",
	"update_rate": 3600000,
	"includes": [ "*.doc" , "*.xls", "*.pdf" ]
  },
  "index": {
  	"index": "mydocs",
  	"type": "doc",
  	"bulk_size": 50
  }
}'
```

Indexing using SSH
------------------

You can now index files remotely using SSH:

* FS URL: `/tmp3`
* Server: `mynode.mydomain.com`
* Username: `username`
* Password: `password`
* Protocol: `ssh` (default to `local`)
* Update Rate: every hour (60 * 60 * 1000 = 3600000 ms)
* Get only docs like `*.doc`, `*.xls` and `*.pdf`

```sh
curl -XPUT 'localhost:9200/_river/mysshriver/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp3",
	"server": "mynode.mydomain.com",
	"username": "username",
	"password": "password",
	"protocol": "ssh",
	"update_rate": 3600000,
	"includes": [ "*.doc" , "*.xls", "*.pdf" ]
  }
}'
```

Searching for docs
------------------

This is a common use case in elasticsearch, we want to search for something ;-)

```sh
curl -XGET http://localhost:9200/docs/doc/_search -d '{
  "query" : {
    "match" : {
        "_all" : "I am searching for something !"
    }
  }
}'
```

Indexing JSon docs
------------------

If you want to index JSon files directly without parsing them through the attachment mapper plugin, you
can set `json_support` to `true`.

```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 3600000,
	"json_support" : true
  },
  "index": {
    "index": "mydocs",
    "type": "doc",
    "bulk_size": 50
  }
}'
```

Of course, if you did not define a mapping prior creating the river, Elasticsearch will auto guess the mapping.

If you have more than one type, create as many rivers as types:

```sh
curl -XPUT 'localhost:9200/_river/mydocs1/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp/type1",
	"update_rate": 3600000,
	"json_support" : true
  },
  "index": {
    "index": "mydocs",
    "type": "type1",
    "bulk_size": 50
  }
}'

curl -XPUT 'localhost:9200/_river/mydocs2/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp/type2",
	"update_rate": 3600000,
	"json_support" : true
  },
  "index": {
    "index": "mydocs",
    "type": "type2",
    "bulk_size": 50
  }
}'
```

You can also index many types from one single dir using two rivers on the same dir and by setting
`includes` parameter:

```sh
curl -XPUT 'localhost:9200/_river/mydocs1/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 3600000,
    "includes": [ "type1*.json" ],
	"json_support" : true
  },
  "index": {
    "index": "mydocs",
    "type": "type1",
    "bulk_size": 50
  }
}'

curl -XPUT 'localhost:9200/_river/mydocs2/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 3600000,
    "includes": [ "type2*.json" ],
	"json_support" : true
  },
  "index": {
    "index": "mydocs",
    "type": "type2",
    "bulk_size: 50
  }
}'
```

Please note that the document `_id` is always generated (hash value) from the JSon filename to avoid issues with
special characters in filename.
You can force to use the `_id` to be the filename using `filename_as_id` attribute:

```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 3600000,
	"json_support": true,
	"filename_as_id": true
  },
  "index": {
    "index": "mydocs",
    "type": "doc",
    "bulk_size": 50
  }
}'
```

Disabling file size field
-------------------------

By default, FSRiver will create a field to store the original file size in octet.
You can disable it using `add_filesize' option:

```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"add_filesize": false
  }
}'
```

Suspend or restart a file river
-------------------------------

If you need to stop a river, you can call the `_stop' endpoint:

```sh
curl 'localhost:9200/_river/mydocs/_stop'
```

To restart the river from the previous point, just call `_start` end point:

```sh
curl 'localhost:9200/_river/mydocs/_start'
```

Ignore deleted files
--------------------

If you don't want to remove indexed documents when you remove a file or a directory, you can
set `remove_deleted` to `false` (default to `true`):


```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"remove_deleted": false
  }
}'
```

Advanced
========

Autogenerated mapping
---------------------

When the FSRiver detect a new type, it creates automatically a mapping for this type.

```javascript
{
  "doc" : {
    "properties" : {
      "content" : {
        "type" : "string",
        "store" : "yes"
      },
      "meta" : {
        "properties" : {
          "author" : {
              "type" : "string",
              "store" : "yes"
          },
          "title" : {
              "type" : "string",
              "store" : "yes"
          },
          "date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "keywords" : {
              "type" : "string",
              "store" : "yes"
          }
        }
      },
      "file" : {
        "properties" : {
          "content_type" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "last_modified" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "indexing_date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "filesize" : {
              "type" : "long",
              "store" : "yes"
          },
          "indexed_chars" : {
              "type" : "long",
              "store" : "yes"
          },
          "filename" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "url" : {
              "type" : "string",
              "store" : "yes",
              "index" : "no"
          }
        }
      },
      "path" : {
        "properties" : {
          "encoded" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "virtual" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "root" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "real" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          }
        }
      }
    }
  }
}
```

Creating your own mapping (analyzers)
-------------------------------------

If you want to define your own mapping to set analyzers for example, you can push the mapping **before** starting the FS River.

```sh
# Create index
$ curl -XPUT "http://localhost:9200/docs/"

# Create the mapping
$ curl -XPUT "http://localhost:9200/docs/doc/_mapping" -d '{
  "doc" : {
    "properties" : {
      "content" : {
        "type" : "string",
        "store" : "yes",
        "analyzer" : "french"
      },
      "meta" : {
        "properties" : {
          "author" : {
              "type" : "string",
              "store" : "yes"
          },
          "title" : {
              "type" : "string",
              "store" : "yes"
          },
          "date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "keywords" : {
              "type" : "string",
              "store" : "yes"
          }
        }
      },
      "file" : {
        "properties" : {
          "content_type" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "last_modified" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "indexing_date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "filesize" : {
              "type" : "long",
              "store" : "yes"
          },
          "indexed_chars" : {
              "type" : "long",
              "store" : "yes"
          },
          "filename" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "url" : {
              "type" : "string",
              "store" : "yes",
              "index" : "no"
          }
        }
      },
      "path" : {
        "properties" : {
          "encoded" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "virtual" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "root" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "real" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          }
        }
      }
    }
  }
}'
```

Generated fields
----------------

FS River creates the following fields :

|   Field (>= 0.4.0)   |   Field (< 0.4.0)    |                Description                  |                    Example                  |
|----------------------|----------------------|---------------------------------------------|---------------------------------------------|
| `content`            | `file.file`          | Extracted content                           | `"This is my text!"`                        |
| `attachment`         | `file`               | BASE64 encoded binary file                  | BASE64 Encoded document                     |
| `meta.author`        | `file.author`        | Author if any in document metadata          | `"David Pilato"`                            |
| `meta.title`         | `file.title`         | Title if any in document metadata           | `"My document title"`                       |
| `meta.date`          |                      | Document date if any in document metadata   | `"2013-04-04T15:21:35"`                     |
| `meta.keywords`      |                      | Keywords if any in document metadata        | `["river","fs","elasticsearch"]`            |
| `file.content_type`  | `file.content_type`  | Content Type                                | `"application/vnd.oasis.opendocument.text"` |
| `file.last_modified` |                      | Last modification date                      | `1386855978000`                             |
| `file.indexing_date` | `postDate`           | Indexing date                               | `"2013-12-12T13:50:58.758Z"`                |
| `file.filesize`      | `filesize`           | File size in bytes                          | `1256362`                                   |
| `file.indexed_chars` | `file.indexed_chars` | Extracted chars if `fs.indexed_chars` > 0   | `100000`                                    |
| `file.filename`      | `name`               | Original file name                          | `"mydocument.pdf"`                          |
| `file.url`           |                      | Original file url                           | `"file://tmp/mydir/otherdir/mydocument.pdf"`|
| `path.encoded`       | `pathEncoded`        | BASE64 encoded file path (for internal use) | `"112aed83738239dbfe4485f024cd4ce1"`        |
| `path.virtual`       | `virtualpath`        | Relative path from root path                | `"mydir/otherdir"`                          |
| `path.root`          | `rootpath`           | BASE64 encoded root path (for internal use) | `"112aed83738239dbfe4485f024cd4ce1"`        |
| `path.real`          |                      | Actual real path name                       | `"/tmp/mydir/otherdir/mydocument.pdf"`      |

Here is a typical JSON document generated by the river:

```javascript
{
   "file":{
      "filename":"test.odt",
      "last_modified":1386855978000,
      "indexing_date":"2013-12-12T13:50:58.758Z",
      "content_type":"application/vnd.oasis.opendocument.text",
      "url":"file:///tmp/testfs_metadata/test.odt",
      "indexed_chars":100000,
      "filesize":8355
   },
   "path":{
      "encoded":"bceb3913f6d793e915beb70a4735592",
      "root":"bceb3913f6d793e915beb70a4735592",
      "virtual":"",
      "real":"/tmp/testfs_metadata/test.odt"
   },
   "meta":{
      "author":"David Pilato",
      "title":"Mon titre",
      "date":"2013-04-04T15:21:35",
      "keywords":[
         "fs",
         "elasticsearch",
         "river"
      ]
   },
   "content":"Bonjour David\n\n\n"
}
```


Advanced search
---------------

You can use meta fields to perform search on.

```sh
curl -XGET http://localhost:9200/docs/doc/_search -d '{
  "query" : {
    "term" : {
        "file.filename" : "mydocument.pdf"
    }
  }
}'
```

Disabling _source
-----------------

If you don't need to highlight your search responses nor need to get back the original file from
Elasticsearch, you can think about disabling `_source` field.

In that case, you need to store `file.filename` field. Otherwise, FSRiver won't be able to remove documents when
they disappear from your hard drive.

```javascript
{
  "doc" : {
    "_source" : { "enabled" : false },
    "properties" : {
      "content" : {
        "type" : "string",
        "store" : "yes"
      },
      "meta" : {
        "properties" : {
          "author" : {
              "type" : "string",
              "store" : "yes"
          },
          "title" : {
              "type" : "string",
              "store" : "yes"
          },
          "date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "keywords" : {
              "type" : "string",
              "store" : "yes"
          }
        }
      },
      "file" : {
        "properties" : {
          "content_type" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "last_modified" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "indexing_date" : {
              "type" : "date",
              "format" : "dateOptionalTime",
              "store" : "yes"
          },
          "filesize" : {
              "type" : "long",
              "store" : "yes"
          },
          "indexed_chars" : {
              "type" : "long",
              "store" : "yes"
          },
          "filename" : {
              "type" : "string",
              "analyzer" : "simple",
              "store" : "yes"
          },
          "url" : {
              "type" : "string",
              "store" : "yes",
              "index" : "no"
          }
        }
      },
      "path" : {
        "properties" : {
          "encoded" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "virtual" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "root" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          },
          "real" : {
              "type" : "string",
              "store" : "yes",
              "index" : "not_analyzed"
          }
        }
      }
    }
  }
}
```

Storing binary source document (BASE64 encoded)
-----------------------------------------------

You can store in elasticsearch itself the binary document using `store_source` option:

```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
	"url": "/tmp",
	"update_rate": 3600000,
	"store_source": true
  },
  "index": {
    "index": "mydocs",
    "type": "doc",
    "bulk_size": 50
  }
}'
```

In that case, a new stored field named `attachment` is added to the generated JSon document.
If you let FSRiver generates the mapping, FSRiver will exclude `attachment` field from
`_source` to save some disk space.

That means you need to ask for field `attachment` when querying:

```sh
curl -XPOST http://localhost:9200/mydocs/doc/_search -d '{
  "fields" : ["attachment", "_source"],
  "query":{
    "match_all" : {}
  }
}'
```

Default generated mapping in this case is:

```javascript
{
  "doc" : {
    "_source" : {
      "excludes" : [ "attachment" ]
    },
    "properties" : {
      "attachment" : {
        "type" : "binary"
      },
      ... // Other properties here
    }
  }
}
```

You can force not to store `attachment` field and keep `attachment` in `_source`:

```sh
# Create index
$ curl -XPUT "http://localhost:9200/docs/"

# Create the mapping
$ curl -XPUT "http://localhost:9200/docs/doc/_mapping" -d '{
  "doc" : {
    "properties" : {
      "attachment" : {
        "type" : "binary",
        "store" : "no"
      },
      ... // Other properties here
    }
  }
}
```

Extracted characters
--------------------

By default FSRiver will extract only a limited size of characters (100000).
But, you can set `indexed_chars` to `1` in FSRiver definition.

```sh
curl -XPUT 'localhost:9200/_river/mydocs/_meta' -d '{
  "type": "fs",
  "fs": {
    "url": "/tmp",
    "indexed_chars": 1
  }
}'
```

That option will add a special field `_indexed_chars` to the document. It will be set to the filesize.
This field is used by mapper attachment plugin to define the number of extracted characters.

Setting `indexed_chars : x` will compute file size, multiply it with x and pass it to Tika using `_indexed_chars` field.

That means that a value of 0.8 will extract 20% less characters than the file size. A value of 1.5 will extract 50% more
characters than the filesize (think compressed files). A value of 1, will extract exactly the filesize.

Note that Tika requires to allocate in memory a data structure to extract text. Setting `indexed_chars` to a high
number will require more memory!


Migrating from version < 0.4.0
==============================

Some important changes have been done in FSRiver 0.4.0:

* You don't have to add attachment plugin anymore as we directly rely on Apache Tika.
* Fields have changed. You should look at [Generated Fields](#generated-fields) section
to know how the old fields have been renamed.

License
=======

```
This software is licensed under the Apache 2 license, quoted below.

Copyright 2011-2012 David Pilato

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
```
