# Triple Redaction Example

This project shows some examples of how you can apply rules to mask the subjects, predicates and objects of RDF triples stored in MarkLogic.

## Redaction Rules
In the [redactionRules](src/main/ml-schemas/redactionRules) folder, you will find rules for redacting the subject, predicate and object separately. Depending on how you want to redaction to work, you can apply different rules when exporting your RDF triples.

The `collections.properties` file is a special file that controls what collections the redaction rules are added to. This allows you to create different sets of rules that can be applied to different content as needed. It is currently configured to put rules into the `security-rules` collection which is used in the example export task.

The following table describes the rule files that can be applied and what collection they below to. The rules don't cover all the possible scenarios and data types. They are just meant to be examples to use as a starting point.

|File|Description|Collection|
| ----------- | ----------- | ----------- |
|subject.json|Applies a deterministic hash of all of the `sem:triple/sem:subject` elements to a 64 character string. Additional randomness can be added to the masking by setting the `salt` property. |security-rules|
|predicate.json|Applies a deterministic hash of all of the `sem:triple/sem:predicate` elements to a 64 character string. Additional randomness can be added to the masking by setting the `salt` property.|security-rules|
|object.json|Applies a deterministic hash of all of the `sem:triple/sem:object` elements (with either no `datatype` or a `datatype` attribute that is something other than the rules listed below) to a 64 character string. Additional randomness can be added to the masking by setting the `salt` property.|security-rules|
|object-decimal.json|Applies a number-specific redaction process to `sem:triple/sem:object` elements with the `datatype` attribute equal to `http://www.w3.org/2001/XMLSchema#decimal`|security-rules|
|object-integer.json|Applies a number-specific redaction process to `sem:triple/sem:object` elements with the `datatype` attribute equal to `http://www.w3.org/2001/XMLSchema#integer`|security-rules|
|object-datetime.json|Applies a date/time-specific redaction process to `sem:triple/sem:object` elements with the `datatype` attribute equal to `http://www.w3.org/2001/XMLSchema#dateTime`|security-rules|
|object-integer-masked.json|Applies a deterministic hash to `sem:triple/sem:object` elements with the `datatype` attribute equal to `http://www.w3.org/2001/XMLSchema#integer`. The output will be a 10 digit numeric value.|Not used|

See the following for specifics on MarkLogic redaction, additional properties you can set for the rules, and how the rules are applied:
1. [Redacting Document Content](https://docs.marklogic.com/guide/app-dev/redaction#chapter)
1. [No Guaranteed Ordering of Rules](https://docs.marklogic.com/guide/app-dev/redaction#id_61575)

## Installing the Rules
This project is meant to run against an existing project you have setup in MarkLogic in which you have loaded some triple data. By default, it runs against the `Documents` database and the rules are installed into the `Schemas` database. You can control all of that through the `gradle.properties` file though.

To install the rules, run (use gradlew.bat on Windows)
```./gradlew mlLoadSchemas```
This loads the redaction rules into your schemas database.

Note: The `mlLoadSchemas` gradle task will not delete rules if you remove them from your directory so if you add and remove rules from the directory or change the collections, you may need to clear and reload your schemas database using the `mlReloadSchemas`. _Be aware that this will clear other artifacts that you may have loaded in your schemas database through other processes or projects so be careful when doing so!_


To export your triples, run
```./gradlew exportTriplesRedacted```
This runs an [mlcp](https://developer.marklogic.com/products/mlcp) process to export all the triples in your `Documents` database to a compressed archive in the `data/export` folder. It applies all the redaction rules that have been added to the `security-rules` collection as controlled by the [collections.properties](src/main/ml-schemas/redactionRules/collections.properties) file. See the `exportTriplesRedacted` task in `build.gradle` for details on the parameters passed to mlcp.


