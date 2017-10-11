# Using the Digitalt Museum API
- [Introduction](#)
- [Querying the Solr index](#solr)
- [Example Solr queries](#examples)
- [Full object view](#full)
- [Retrieving media](#media)

This is an overview of methods to query the Digitalt Museum (DiMu) API. Field use and detail level may vary in the different collections/museums.

Digitalt Museum contains more than 3,860,000 heritage objects from [Norway](http://www.digitaltmuseum.no) (1,85 mill. objects from 174 museums) and [Sweden](http://www.digitaltmuseum.se) (2 mill. objects from 51 museums) (27.09.2016).

Digitalt Museum and API to the metadata is provided by [KulturIT](http://www.kulturit.no). A general API documentation (in Norwegian only) is at http://api.dimu.org/doc/public_api.html. The documentation below is based on KulturIT's documentation, on Nasjonalmuseet's experience of using the API to create web applications and on the Nordic Museum’s work with Wikimedia Commons uploads during the fall of 2017.

The API endpoint is `http://api.dimu.org/`

<h2>Digitalt Museum: museums and collections</h2>

An xml overview of museums or collections per country can be found like this:

```
/api/owners?country=no&api.key=demo
/api/owners?country=se&api.key=demo
```

The term `owner` here corresponds to a museum or collection. Each owner has an element `identifier` which is used to limit an API search. The `owner` structure is flat, but owners that are children of other owners in the hiearchy have a `parent` element. Calls to the API should always be limited to one or more owners unless you specifically need to query the entire index.

The Nordic Museum’s owner code is `S-NM`.

<a name="solr"></a><h2>Querying the Digitalt Museum Solr index</h2>

For queries to the Digitalt Museum Solr index most common parameters can be used, including paging, sorting and to a certain extent faceting. 

<h3>Fields</h3>

There are several searchable fields in the Solr index. Some are stored and not tokenized, and some fields are tokenized but not stored, but still searchable. Typical examples of stored fields are the `ingress` fields. The content of these fields are displayed in the response. There are other stored fields that are not part of `ingress` that can be be used.

``` javascript
{
  artifact.ingress.title: "Stemma. Quattrocento [Maleri]",
  artifact.ingress.producer: "Slaattelid, Mari",
  artifact.uuid: "5A4FCFDD-CB15-4DB1-971F-4FF5D15E68F6",
  artifact.pictureCount: 1,
  artifact.defaultPictureIndex: 15618,
  artifact.childCount: 2,
  artifact.defaultMediaIdentifier: "012wWWHgNJLA",
  artifact.hasPictures: true,
  artifact.uniqueId: "021045839502",
  artifact.ingress.production.toYear: 1999,
  artifact.ingress.producerRole: "Kunstner",
  artifact.publishedDate: "2015-04-18T06:18:23.538Z",
  artifact.ingress.license: [
    "AC 1"
    ],
  identifier.id: "MS-04266-1999",
  artifact.hasChildren: true,
  identifier.owner: "NMK-B",
  artifact.type: "Fineart",
  artifact.ingress.production.fromYear: 1999,
  artifact.ingress.subjects: [
    "Bildende kunst"
    ],
  artifact.updatedDate: "2015-09-22T07:49:19.08Z"
}
```
<h4>Response formats</h4>

The response will be in xml by default. Add the parameter `wt=json` for a JSON response.

<h4>Indexed fields</h4>

| Field name | Description |
| ---------- | ------------ |
| *identifier.id* | Museum or collection's own identifier / inventory no., e.g. NMK.2005.0257 |
| *identifier.owner* | Museum or collection code, e.g. NMK-B |
| *artifact.uniqueId* | DiMu-specific unique id for object, e.g. 021045835852 |
| *artifact.folderUids* | Folder ID of object, i.e. find all objects in a folder |
| *artifact.name* | Name, alternative name, etc | 
| *artifact.type* | Type of object, e.g. 'Exhibition' |
|* artifact.classification* | Classification. Nasjonalmuseet uses [Outline](http://kulturnav.org/a8797483-ff02-4a4c-adf1-b406cbcd6fc2) where relevant. |
|*artifact.pictureCount* | No. of images for this object |
|*artifact.hasPictures* | *true* or *false* |
|*artifact.defaultMediaIdentifier* | Id for default picture on DiMu media server (DMS) |
|~~artifact.defaultPictureIndex~~ | Id for default picture (obsolete) |
| *artifact.publishedDate* | Date published to DiMu |
| *artifact.updatedDate* | Last updated |
| *artifact.ingress.title* or *artifact.title* | Object title |
| *artifact.ingress.producer* or *artifact.producer* | Producer |
| *artifact.ingress.producerRole* | Role code for producer: Artist, designer, architect etc |
| *artifact.ingress.additionalProducers* | Any additional producers |
| *artifact.ingress.production.fromYear* or *artifact.event.fromYear* | Production year start |
| *artifact.ingress.production.toYear* or *artifact.event.toYear* | Production year end |
| *artifact.ingress.production.place* or *artifact.event.place* | Production place |
| *artifact.eventDescription* | Event description |
| *artifact.ingress.classification* | Classification |
| *artifact.ingress.subjects* | Subjects |
| *artifact.depictedPerson* | Depicted person |
| *artifact.depictedPlace* | Depicted place |
| *artifact.material* | Material |
| *artifact.technique* | Technique |
| *artifact.ingress.license* or *artifact.license* | Object license (not picture license) |
| *allContent* | Free text search field (default) |

<a name="examples"></a>
<h2>Example Solr queries</h2>

Searching in fields is done by adding `field:query` as a value to the `fq` parameter. Multiple searches can be joined by AND or OR separators or by stacking multiple fq parameters. Multiple fq parameters count as an AND search.

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:S-NM*&wt=json&api.key=demo

The above query will return all Nordic Museum objects in the DiMu API. Mandatory parameters for a query are the `q` parameter and an api key. The "demo" key will only return 10 rows.

<b>Objects related to a place [*Oslo*]:</b>

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.event.place:Oslo&wt=json&api.key=demo

<b>Object name [*Maleri* (painting)]:</b>

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.name:Maleri&wt=json&api.key=demo

<b>Object name [*Ukiyo-e*]:</b>

With a specified number of rows returned per request. 100 rows is maximum. The response can be paged with the `start` parameter.

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.name:Ukiyo-e&wt=json&rows=100&api.key=demo

<b>Object type [*Exhibition*]:</b>

With a specified number of rows returned per request. 100 rows is maximum. The response can be paged with the `start` parameter.

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.type:Exhibition&wt=json&rows=100&api.key=demo

<b>Objects by a specific artist/producer [*Harriet Backer*]:</b>

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.producer:Harriet%20Backer&wt=json&api.key=demo

<b>Facet by producer, ordered alphabetically (default):</b>

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.hasPictures:true&facet=true&facet.mincount=1&facet.field=artifact.ingress.producer&facet.limit=7000&facet.sort=index&wt=json&rows=0&api.key=demo

<b>Facet by producer, ordered by frequency / number of objects in collection:</b>

http://api.dimu.org/api/solr/select?q=*&fq=identifier.owner:NMK*&fq=artifact.hasPictures:true&facet=true&facet.mincount=1&facet.field=artifact.ingress.producer&facet.limit=7000&wt=json&facet.sort=count&rows=0&api.key=demo

<a name="full"></a><h2>Full object view</h2>
The Solr response has limitations in certain fields. For instance, it will only return one title for an object although multiple titles may exist, for instance titles in other languages. Also, the Solr response will only list one producer per object even if more exist, and it will not list any attribute associated with the production event. E.g. if a production event carries the attribute <em>antatt sikker</em> (<em>presumed certain</em>), it will not be present in the Solr response.. 

The full object view (in JSON) of individual objects can be requested by using the object’s `artifact.uuid` from the Solr response:
```
http://api.dimu.org/artifact/uuid/3DF10C96-B33B-45C1-92BF-D9211CE574C8
```
The full view will return more fields than the Solr response, in particular it will have a more elaborate description of the production event.
[Se here for an overview of the full object representation](full-object-view.md).

<a name="media"></a><h2>Retrieving media</h2>
[See here for how to retrieve pictures from the DiMu media servers](retrieving-media.md).
