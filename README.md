# Python/Clojure cheatsheet

This document is intended for Python programmers who want to know
how to do various common things in Clojure.

Almost all of the examples below are intended to be pasted in a Python REPL or a Clojure REPL.
The Python version always precedes the Clojure version.

If this is missing some common task, please file an issue.


## Tasks


### Getting a REPL started

#### Python

On Linux or OS X, run "python" in a terminal.

On Windows, install Python 2.x; start cmd; run "C:\Python27\python"


#### Clojure

On all platforms, install [leiningen](http://leiningen.org/)
```
~/bin/lein new dummyproj
cd dummyproj
lein repl
```



### Print something with a newline

```python
print "success", 111, "another string"
```

```clojure
(println "success" 111 "another string")
```



### Print something without a newline

```python
import sys
sys.stdout.write("hello world")
```

```clojure
(print "hello world")
```



### Print a representation of an object

```python
print repr(["hello world", 3])
```

```clojure
(prn ["hello world" 3])
```



### Pretty-print an object

```python
import pprint
pprint.pprint(list(range(10) for n in range(10)))
```

```clojure
(require '[clojure.pprint :refer [pprint]])
(pprint (for [_ (range 10)] (range 10)))
```

or using [fipp](https://github.com/brandonbloom/fipp) for better pretty-printing in some cases:

```clojure
; with [fipp "0.4.1"] in :dependencies in your project.clj

(require '[fipp.edn :refer (pprint) :rename {pprint fipp}])
(fipp (for [_ (range 10)] (range 10)))
```



### Get a reversed list (Python) or vector (Clojure)

```python
list(reversed([1, 2, 3]))
```

```clojure
(vec (reverse [1 2 3]))
```



### Assert something

```python
assert 1 == 1, "1 is not equal to 1"
assert 1 == 2, "1 is not equal to 2"
```

```clojure
(assert (= 1 1) "1 is not equal to 1")
(assert (= 1 2) "1 is not equal to 2")
```



### Make sure a directory exists for a file, creating it if it doesn't.

```python
import os
try:
	os.makedirs(os.path.dirname("foo2/bar/whatever.txt"))
except OSError:
	pass
```

```clojure
(require '[clojure.java.io :refer [as-file]])
(.mkdirs (.getParentFile (as-file "foo/bar/whatever.txt")))
```



### Get the response body for an HTTP request as a string, using language/platform-native facilities.

```python
import urllib2
urllib2.urlopen("http://www.google.com/").read()
```

```clojure
(require '[clojure.java.io :refer [as-url]])
(slurp (as-url "http://www.google.com/"))
```



### Print out a file, along with line numbers.

```python
with open("/etc/passwd") as f:
	for n, line in enumerate(f):
		print n, line.rstrip("\r\n")
```

```clojure
(require '[clojure.java.io :refer [reader]])
(with-open [rdr (reader "/etc/passwd")]
  (doseq [[n line] (map-indexed vector (line-seq rdr))]
    (println n line)))
```



### Write a new file

```python
with open("deleteme-1", "wb") as f:
	f.write("contents")
```

```clojure
(spit "deleteme-1" "contents")
```



### Append a line to a file

```python
with open("deleteme-2", "ab") as f:
	f.write("another line\n")
```

```clojure
(require '[clojure.java.io :refer [writer]])
(with-open [wrtr (writer "deleteme-2" :append true)]
  (.write wrtr "another line\n"))
```



### Load the lines of a file into a set

```python
with open("/etc/passwd") as f:
	set(line.rstrip("\r\n") for line in f)
```

```clojure
(require '[clojure.java.io :refer [reader]])
(with-open [rdr (reader "/etc/passwd")]
  (set (line-seq rdr)))
```



### Parse some JSON in a string

```python
import json
json.loads('[{"a": 10.1, "b": [true, false, null]}, 1]')
```

```clojure
; with [org.clojure/data.json "0.2.4"] in :dependencies in your project.clj

(require '[clojure.data.json :as json])
(json/read-str "[{\"a\": 10.1, \"b\": [true, false, null]}, 1]")
```



### Encode some data to JSON and write it to a file

```python
import json
with open("/tmp/json", "wb") as f:
	json.dump([{"a": 10.1, "b</script>": [True, False, None]}, 1], f)
```

with [data.json](https://github.com/clojure/data.json):

```clojure
; with [org.clojure/data.json "0.2.4"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [writer]])
(require '[clojure.data.json :as json])
(with-open [wrtr (writer "/tmp/json")]
  (json/write [{"a" 10.1 "b</script>" [true false nil]} 1] wrtr))
; unlike Python, escapes forward slash by default; also emits no whitespace
```

with [cheshire](https://github.com/dakrone/cheshire):

```clojure
; with [cheshire "5.3.1"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [writer]])
(require '[cheshire.core :as cheshire])
(with-open [wrtr (writer "/tmp/json")]
  (cheshire/generate-stream [{"a" 10.1 "b</script>" [true false nil]} 1] wrtr))
; like Python, does not escape forward slash by default; unlike Python, emits no whitespace
```

<!--
https://github.com/mmcgrana/clj-json/
https://github.com/michalmarczyk/clj-lazy-json
-->



### Parse some JSON in a file

```python
import json
with open("/tmp/json", "rb") as f:
	json.load(f)
```

with data.json:

```clojure
; with [org.clojure/data.json "0.2.4"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [reader]])
(require '[clojure.data.json :as json])
(with-open [rdr (reader "/tmp/json")]
  (json/read rdr))
```

with cheshire:

```clojure
; with [cheshire "5.3.1"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [reader]])
(require '[cheshire.core :as cheshire])
(with-open [rdr (reader "/tmp/json")]
  (first (cheshire/parsed-seq rdr)))
```



### Write some EDN to a file

```python
# https://github.com/swaroopch/edn_format
import edn_format

with open("/tmp/edn", "wb") as f:
  obj = {"string-key": [3, 4], edn_format.Keyword("keyword-key"): {1, 2, 3}}
  f.write(edn_format.dumps(obj))
```

```clojure
(require '[clojure.java.io :refer [writer]])
(with-open [wrtr (writer "/tmp/edn")]
  (binding [*out* wrtr]
    (pr {"string-key" [3 4] :keyword-key #{1 2 3}})))
```



### Parse some EDN in a file and get a keyword key

```python
# https://github.com/swaroopch/edn_format
import edn_format

with open("/tmp/edn", "rb") as f:
  # Note: edn_format cannot read EDN where a key is a map or set or vector
  # Use https://github.com/dreid/edn in the future
  obj = edn_format.loads(f.read())
  print obj
  print obj[edn_format.Keyword("keyword-key")]
```

```clojure
(require '[clojure.edn :as edn])
(require '[clojure.java.io :refer [reader]])
(with-open [rdr (java.io.PushbackReader. (reader "/tmp/edn"))]
  (let [obj (edn/read rdr)]
    (println obj)
    (println (:keyword-key obj))))
```



### Read lines from stdin, write out unique lines to stdout

```python
#!/usr/bin/python

import sys

s = set()

for line in sys.stdin:
        s.add(line.rstrip("\r\n"))

for item in s:
        print item
```

```clojure
; run with: lein run -m yournamespace

(defn -main []
  (let [rdr (java.io.BufferedReader. *in*)]
    (doseq [item (set (line-seq rdr))]
      (println item))))
```


<!-- TODO Log a message: -->



### Make a string with 80 asterisks

```python
"*"*80
```

```clojure
(require '[clojure.string :as str])
(str/join "" (repeat 80 "*"))
```



### Make a list (Python) or vector (Clojure) with 10 copies of a string

```python
["hi"] * 10
```

```clojure
(vec (repeat 10 "hi"))
```



### Replace all instances of X in string Y

```python
"hello".replace("l", "x")
```

```clojure
(require '[clojure.string :as str])
(str/replace "hello" "l" "x")
```



### Replace first instance of X in string Y

```python
"hello".replace("l", "x", 1)
```

```clojure
(require '[clojure.string :as str])
(str/replace-first "hello" "l" "x")
```



### Join a list (Python) or vector (Clojure) with `", "`

```python
", ".join(["a", "bee", "see"])
```

```clojure
(require '[clojure.string :as str])
(str/join ", " ["a" "bee" "see"])
```


### Lowercase all strings in a list (Python) or vector (Clojure)

```python
list(s.lower() for s in ["Mixed", "CASE"])
```

```clojure
(require '[clojure.string :as str])
(mapv str/lower-case ["Mixed" "CASE"])
```



### Remove whitespace from both ends of a string

```python
" hello  \r\n ".strip()
```

```clojure
(require '[clojure.string :as str])
(str/trim " hello  \r\n ")
```



### Split a string on space

```python
"hello  world test  ".split(" ")
# returns ['hello', '', 'world', 'test', '', '']

"hello  world test  ".split()   
# returns ['hello', 'world', 'test']
```

```clojure
(require '[clojure.string :as str])

(str/split "hello  world test  " #" ")
; returns ["hello" "" "world" "test"]

(str/split "hello  world test  " #" +")
; returns ["hello" "world" "test"]
```



### Define and call a function with an optional argument

```python
def optional_arg(a, b=10):
	"""
	Documentation
	"""
	return a + b

optional_arg(100)
optional_arg(100, 1)
```

```clojure
(defn optional-arg
  "Documentation"
  ([a] (optional-arg a 10))
  ([a b] (+ a b)))

(optional-arg 100)
(optional-arg 100 1)
```

Do *not* do this, as the function does not complain when you pass too many arguments:

```clojure
(defn optional-arg
  "Documentation"
  [a & [b]]
  (+ a (or b 10)))

(optional-arg 100)
(optional-arg 100 1)
(optional-arg 100 1 2)
```


### Change some values in a map

```python
record = {"name": {"first": "Kim", "last": "Schmitz"}, "age": 31, "secret": "delete-me"}
new_record = record.copy()
new_record["name"]["last"] = "Dotcom"
new_record["age"] += 1
del new_record["secret"]
print new_record
```

```clojure
(let [record     {:name {:first "Kim", :last "Schmitz"} :age 31 :secret "delete-me"}
      new-record (-> record
                     (assoc-in [:name :last] "Dotcom")
                     (dissoc :secret)
                     (update-in [:age] inc))]
  (prn new-record))
```



### Get the difference of two sets

```python
set([1, 2, 3, 4]) - set([2, 3, 10])
```

```clojure
(require '[clojure.set :as set])
(set/difference #{1 2 3 4} #{2 3 10})
```



### Get the intersection of two sets

```python
set([1, 2, 3, 4]) & set([2, 3, 10])
```

```clojure
(require '[clojure.set :as set])
(set/intersection #{1 2 3 4} #{2 3 10})
```



### Get the union of two sets

```python
set([1, 2, 3, 4]) | set([2, 3, 10])
```

```clojure
(require '[clojure.set :as set])
(set/union #{1 2 3 4} #{2 3 10})
```



### Get milliseconds since epoch

```python
time.time() * 1000
# returns a float
```

```clojure
(System/currentTimeMillis)
; returns a Long
```



### Run a process and get the output

```python
import subprocess
subprocess.check_output(["ls", "-l"])
# nonexistent binary -> OSError raised
# non-zero exit -> CalledProcessError raised
```

```clojure
(require '[clojure.java.shell :refer [sh]])
(sh "ls" "-l")
; output is a map with :out string, :err string, and :exit code
; nonexistent binary -> IOException thrown
; non-zero exit -> non-zero :exit value in return map
```



## Other Clojure/Leiningen tips


### Make `lein repl`, `lein run`, and other project commands run at full JVM performance; not optimized for startup time

Add
```clojure
:jvm-opts ^:replace ["-server"]
```

to `project.clj`



### Make `lein repl`, `lein run`, and other project commands run at full JVM performance, and give JVM 2GB of heap

Add
```clojure
:jvm-opts ^:replace ["-server" "-Xmx2G"]
```

to `project.clj`



### Make `lein repl` not slow when the project is on a network drive

Add

```clojure
:repl-options {:history-file nil}
```

to `project.clj`



### Get a ClojureScript REPL managed by austin

```
sudo apt-get install phantomjs
```

Add [austin](https://github.com/cemerick/austin) to your `~/.lein/profiles.clj` like so:

```clojure
{:user {:plugins [[com.cemerick/austin "0.1.3"]]}}
```

Start a Clojure REPL as usual with `lein repl`, then run

```clojure
(cemerick.austin.repls/exec)
```



### Get a ClojureScript REPL powered by your browser

Add [austin](https://github.com/cemerick/austin) to your `~/.lein/profiles.clj` like so:

```clojure
{:user {:plugins [[com.cemerick/austin "0.1.3"]]}}
```

Start a Clojure REPL as usual with `lein repl`, then run

```clojure
(cemerick.piggieback/cljs-repl :repl-env (cemerick.austin/repl-env))
```
