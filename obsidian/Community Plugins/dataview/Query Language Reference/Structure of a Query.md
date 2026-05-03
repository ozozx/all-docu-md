Dataview offers [[DQL JS and Inlines|multiple ways]] to write queries and the syntax differs for each.

This page provides information on how to write a **Dataview Query Language** (**DQL**) query. If you're interested in how to write Inline Queries, refer to the [[DQL JS and Inlines#Inline DQL|inline section on DQL, JS and Inlines]]. You'll find more information about **Javascript Queries** on the [[obsidian/Community Plugins/dataview/JavaScript Reference/Overview|Javascript Reference]].

**DQL** is a SQL like query language for creating different views or calculations on your data. It supports:

- Choosing an **output format** of your output (the [[Query Types|Query Type]])
- Fetch pages **from a certain [[Sources|source]]**, i.e. a tag, folder or link
- **Filtering pages/data** by simple operations on fields, like comparison, existence checks, and so on
- **Transforming fields** for displaying, i.e. with calculations or splitting up multi-value fields
- **Sorting** results based on fields
- **Grouping** results based on fields
- **Limiting** your result count

> [!warning]
>
> If you are familiar with SQL, please read [[Dataview Query Language (DQL) and SQL|Differences to SQL]] to avoid confusing DQL with SQL.

Let's have a look at how we can put DQL to use.

## General Format of a DQL Query

Every query follows the same structure and consists of

- exactly one **Query Type** with zero, one or many [[Adding Metadata|fields]], depending on query type
- zero or one **FROM** data commands with one to many [[Sources|sources]]
- zero to many other **data commands** with one to many [[Expressions|expressions]] and/or other infos depending on the data command

At a high level, a query conforms to the following pattern:

````
```dataview <QUERY-TYPE> <fields>
FROM <source>
<DATA-COMMAND> <expression>
<DATA-COMMAND> <expression>
          ...
```
````

> [!note] Only the Query Type is mandatory.

The following sections will explain the theory in further detail.

## Choose a Output Format

The output format of a query is determined by its **Query Type**. There are four available:

1. **TABLE**: A table of results with one row per result and one or many columns of field data.
2. **LIST**: A bullet point list of pages which match the query. You can output one field for each page alongside their file links.
3. **TASK**: An interactive task list of tasks that match the given query.
4. **CALENDAR**: A calendar view displaying each hit via a dot on its referred date.

The Query Type is the **only mandatory command in a query**. Everything else is optional.

> [!note] Possibly memory intense examples
>
> Depending on the size of your vault, executing the following examples can take long and even freeze Obsidian in extreme cases. It's recommended that you specify a `FROM` to restrict the query execution to a specific subset of your vaults' files. See next section.

````
Lists all pages in your vault as a bullet point list
```dataview
LIST
```

Lists all tasks (completed or not) in your vault
```dataview
TASK
```

Renders a
Calendar view where each page is represented as a dot on its creation date.
```dataview
CALENDAR file.cday
```

Shows a table with all pages of your vault, their field value of due, the files' tags and an average of the values of multi-value field working-hours
```dataview
TABLE due, file.tags AS "tags", average(working-hours)
```
````

> [!info] Read more about the available Query Types and how to use them [[Query Types|here]].

## Choose your source

Additionally to the Query Types, you have several **Data Commands** available that help you restrict, refine, sort or group your query. One of these query commands is the **FROM** statement. `FROM` takes a [[Sources|source]] or a combination of [[Sources|sources]] as an argument and restricts the query to a set of pages that match your source.

It behaves differently from the other Data Commands: You can add **zero or one** `FROM` data command to your query, right after your Query Type. You cannot add multiple FROM statements and you cannot add it after other Data Commands.

````
Lists all pages inside the folder Books and its sub folders
```dataview
LIST
FROM "Books"
```

Lists all pages that include the tag #status/open or #status/wip
```dataview
LIST
FROM #status/open OR #status/wip
```

Lists all pages that have either the tag #assignment and are inside folder "30 School" (or its sub folders), or are inside folder "30 School/32 Homeworks" and are linked on the page School Dashboard Current To Dos
```dataview
LIST
FROM (#assignment AND "30 School") OR ("30 School/32 Homeworks" AND outgoing([[School Dashboard Current To Dos]]))
```
````

> [!info] Read more about `FROM` [[Data Commands#FROM|here]].

## Filter, sort, group or limit results

In addition to the Query Types and the **Data command** `FROM` that's explained above, you have several other **Data Commands** available that help you restrict, refine, sort or group your query results.

All data commands except the `FROM` command can be used **multiple times in any order** (as long as they come after the Query Type and `FROM`, if `FROM` is used at all). They'll be executed in the order they are written.

Available are:

1. **FROM** like explained [[#Choose your source|above]].
2. **WHERE**: Filter notes based on information **inside** notes, the meta data fields.
3. **SORT**: Sorts your results depending on a field and a direction.
4. **GROUP BY**: Bundles up several results into one result row per group.
5. **LIMIT**: Limits the result count of your query to the given number.
6. **FLATTEN**: Splits up one result into multiple results based on a field or calculation.

````
Lists all pages that have a metadata field `due` and where `due` is before today
```dataview
LIST
WHERE due AND due < date(today)
``` 
Lists the 10 most recently created pages in your vault that have the tag #status/open
```dataview
LIST
FROM #status/open
SORT file.ctime DESC
LIMIT 10
``` 
Lists the 10 oldest and incomplete tasks of your vault as an interactive task list, grouped by their containing file and sorted from oldest to newest file.
```dataview
TASK
WHERE !completed
SORT created ASC
LIMIT 10
GROUP BY file.link
SORT rows.file.ctime ASC
```
````

Find out more about available [[Data Commands|data commands]].

## Examples

Following are some example queries. Find more examples [[Examples|here]].

````
```dataview
TASK
```
````

````
```dataview
TABLE recipe-type AS "type", portions, length
FROM #recipes
```
````

````
```dataview
LIST
FROM #assignments
WHERE status = "open"
```
````

````
```dataview
TABLE file.ctime, appointment.type, appointment.time, follow-ups
FROM "30 Protocols/32 Management"
WHERE follow-ups
SORT appointment.time
```
````

````
```dataview
TABLE L.text AS "My lists"
FROM "dailys"
FLATTEN file.lists AS L
WHERE contains(L.author, "Surname")
```
````

````
```dataview
LIST rows.c
WHERE typeof(contacts) = "array" AND contains(contacts, [[Mr. L]])
SORT length(contacts)
FLATTEN contacts as c
SORT link(c).age ASC
```
````
