# Overview #
A Java library supporting the server side for AJAX autocomplete.  This project can be integrated in two ways:
  1. **Standalone server** : A standalone autocomplete HTTP server providing JSON / REST AJAX autocomplete functionality for any language.
  1. **Java library** : A jar providing Autocomplete datastructures that can be easily integrated into an existing Java projects.

The library is build around an <i>autocomplete tree</i> that supports prefix queries on <i>entities</i>.  Information about entities are stored in <i>entries</i> containing the entitiy and a <a>score</a> for the entity.  The score is used to rank and truncate results when there are many entities that match a prefix query.

Queries can match against multiple psuedonyms for the same entity("Obama", "Barack Obama", "Barack Hussain Obama").

# Usage #
### Library Usage ###
The library provides an AutocompleteTree containing AutocompleteEntries.
```
        // Create the autocomplete datastructure
        AutocompleteTree<Integer, City> tree = new AutocompleteTree<Integer, City>();

        // Associate cities with their ids
        tree.add(1, new City(1, "Chicago", "Illinois"));
        tree.add(2, new City(2, "Moline", "Illinois"));
        tree.add(3, new City(3, "Minneapolis", "Minnesota"));
        tree.add(4, new City(4, "St. Paul", "Minnesota"));
        tree.add(5, new City(5, "Boston", "Massachussets"));
        ...

        tree.increment(5);      // increments the score for Boston by 1.
        tree.setScore(3, 9.0);  // sets the score for Minneapolis to 9.0;

        // Returns the top three cities that start with "ch" ordered by score.
        SortedSet<AutocompleteEntry<Integer, City>> results = tree.autocomplete("ch", 3);
        for (AutocompleteEntry<Integer, City> entry : results) {
            System.out.println("city " + entry.getValue() + " with score " + entry.getScore());
        }
```

API details are described on the [AutocompleteTree javadoc](http://autocomplete-server.googlecode.com/svn/trunk/docs/api/edu/macalester/acs/AutocompleteTree.html)


### Server Usage ###

The HTTP server is a standalone service that provides a RESTful JSON interface to a persistent AutocompleteTree.  The tree is persisted through a transaction file that contains information about all entries in the tree.<p>

The tree supports autocomplete features on extensible autocomplete entities represented as (key, value) hashtables.  The hashtable keys must be strings.  The values can be any types supported by json. Three keys are required (id, name, score).<p>

To start up the server, download the zip file, and run the following command from inside it:<br>
<pre><code>java -cp lib/jlhttp.jar:lib/json_simple-1.1.jar:autocomplete-server-0.4.jar \<br>
      edu.macalester.acs.server.AutocompleteServer \<br>
      tx.log \<br>
      8888<br>
</code></pre>
The first argument after the class name is the name of transaction log used to persist the data about autocomplete entities.  The second argument is the port on which the server should listen.<p>

Clients for the service can be easily written in any language using Json.<br>
For example, a python client can be written as:<br>
<pre><code>import httplib<br>
<br>
# update (or create) an entity<br>
conn = httplib.HTTPConnection("localhost", 10101)<br>
conn.request("POST", "/update", '{ "id" : "34a", "name" : "Bob", "score" : 300.2, "foo" : "bar"}')<br>
print conn.getresponse().read()<br>
&gt;&gt;&gt; okay<br>
<br>
# execute an ajax query<br>
conn.request("GET", "/autocomplete?query=b&amp;max=2")<br>
print conn.getresponse().read()<br>
&gt;&gt;&gt; [{"id":"34a","name":"Bob","score":300.2,"foo":"bar"}, {"id":"20395","name":"Myrtle Beach","state":"South Carolina","score":99.9812292931998}] <br>
</code></pre>

More details about the API are provided in the <a href='http://autocomplete-server.googlecode.com/svn/trunk/docs/api/edu/macalester/acs/server/AutocompleteServer.html'>AutocompleteServer javadoc</a>.<br>
<h1>Performance</h1>
<h3>Library Performance</h3>
The <a href='http://code.google.com/p/autocomplete-server/source/browse/trunk/test/edu/macalester/acs/AutocompleteBenchmarker.java'>AutocompleteBenchmarker</a> estimates that the library can perform 50,000 autocomplete queries per second against a dictionary of size 25,000 on a Macbook Pro.<br>
<br>
The lookup performance of the algorithm is O(log(n) + m) where n is the size of the dictionary, and m is the number of matching terms.  The data structure incorporates a cache for short-length queries that are typically associated with a large number of matching terms.<br>
<br>
The memory performance scales linearly with the number of dictionary entries, with somewhere on the order of 100 bytes overhead per dictionary entry.<br>
<h3>Server Performance</h3>

The HTTP server imposes roughly a half millisecond performance penalty on top of all autocomplete instructions.