# Python/Clojure cheatsheet

This document is intended for Python programmers who want to know
how to do various common things in Clojure.

Almost all of the examples below are intended to be pasted in a Python REPL or a Clojure REPL.
The Python version always precedes the Clojure version.

The Python code here is designed to work in both Python 2.7 and Python 3.3+.
The Clojure code here was tested with Clojure 1.6.0.


## Tasks


### Get a REPL started

#### Python

On Linux or OS X, run "python" in a terminal.

On Windows, install Python 2.7 or 3.4; start cmd; run "C:\Python27\python" or "C:\Python34\python"


#### Clojure

On all platforms, install [leiningen](http://leiningen.org/), then
```
~/bin/lein new dummyproj
cd dummyproj
lein repl
```



### Print something with a newline

```python
from __future__ import print_function

print("success", 111, "another string")
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
from __future__ import print_function

print(repr(["hello world", 3]))
```

```clojure
(prn ["hello world" 3])
```



### Pretty-print an object

```python
import pprint
pprint.pprint(list(list(range(10)) for n in range(10)))
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
try:
  import urllib.request as urllib2
except:
  import urllib2

urllib2.urlopen("http://www.google.com/").read()
```

```clojure
(require '[clojure.java.io :refer [as-url]])
(slurp (as-url "http://www.google.com/"))
```



### Print out a file, along with line numbers.

```python
from __future__ import print_function

with open("/etc/passwd") as f:
  for n, line in enumerate(f):
    print(n, line.rstrip("\r\n"))
```

```clojure
(require '[clojure.java.io :refer [reader]])
(with-open [rdr (reader "/etc/passwd")]
  (doseq [[n line] (map-indexed vector (line-seq rdr))]
    (println n line)))
```



### Write a Unicode string to a new file

```python
import codecs

with codecs.open("deleteme-1", "wb", encoding="utf-8") as f:
  f.write(u"contents \ucccc")
```

```clojure
(spit "deleteme-1" "contents \ucccc" :encoding "utf-8")
```



### Append a Unicode string to a file

```python
import codecs

with codecs.open("deleteme-2", "ab", encoding="utf-8") as f:
  f.write(u"another line \ucccc\n")
```

```clojure
(require '[clojure.java.io :refer [writer]])
(with-open [wrtr (writer "deleteme-2" :encoding "UTF-8" :append true)]
  (.write wrtr "another line \ucccc\n"))
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

Python with default `json.dump`:

```python
import codecs
import json

with codecs.open("/tmp/json", "wb", encoding="utf-8") as f:
  json.dump([{u"a \ucccc": 10.1, u"b</script>": [True, False, None]}, 1], f)
```

The file output is `[{"b</script>": [true, false, null], "a \ucccc": 10.1}, 1]`

(whitespace, no forward slash escaping, Unicode as \u escapes)


Python with `json.dump` and `ensure_ascii=False`:

```python
import codecs
import json

with codecs.open("/tmp/json", "wb", encoding="utf-8") as f:
  json.dump([{u"a \ucccc": 10.1, u"b</script>": [True, False, None]}, 1], f, ensure_ascii=False)
```

The file output is `[{"b</script>": [true, false, null], "a 쳌": 10.1}, 1]`

(whitespace, no forward slash escaping, Unicode as UTF-8)


Clojure with [data.json](https://github.com/clojure/data.json):

```clojure
; with [org.clojure/data.json "0.2.4"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [writer]])
(require '[clojure.data.json :as json])
(with-open [wrtr (writer "/tmp/json")]
  (json/write [{"a \ucccc" 10.1 "b</script>" [true false nil]} 1] wrtr))
```

data.json writes `[{"a \ucccc":10.1,"b<\/script>":[true,false,null]},1]`

(no whitespace, yes forward slash escaping, Unicode as \u escapes)


Clojure with [cheshire](https://github.com/dakrone/cheshire):

```clojure
; with [cheshire "5.3.1"] in :dependencies in your project.clj

(require '[clojure.java.io :refer [writer]])
(require '[cheshire.core :as cheshire])
(with-open [wrtr (writer "/tmp/json")]
  (cheshire/generate-stream [{"a \ucccc" 10.1 "b</script>" [true false nil]} 1] wrtr))
```

cheshire writes `[{"a 쳌":10.1,"b</script>":[true,false,null]},1]`

(no whitespace, no forward slash escaping, Unicode as UTF-8)

<!--
https://github.com/mmcgrana/clj-json/
https://github.com/michalmarczyk/clj-lazy-json
-->



### Parse some JSON in a file

```python
import json
with open("/tmp/json", "r") as f:
  json.load(f)
```

Python 2 `json.load` assumes utf-8 encoding; Python 3 `open` assumes utf-8 encoding and `json.load` parses Unicode.

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

The Python version requires https://github.com/swaroopch/edn_format.

```python
import codecs
import edn_format

with codecs.open("/tmp/edn", "wb", encoding="utf-8") as f:
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

The Python version requires https://github.com/swaroopch/edn_format.

```python
from __future__ import print_function
import edn_format

with open("/tmp/edn", "r") as f:
  # Note: edn_format cannot read EDN where a key is a vector
  # Maybe use https://github.com/dreid/edn in the future
  obj = edn_format.loads(f.read())
  print(obj)
  # prints {Keyword(keyword-key): frozenset([1, 2, 3]), 'string-key': [3, 4]}
  print(obj[edn_format.Keyword("keyword-key")])
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

from __future__ import print_function
import sys

s = set()

for line in sys.stdin:
  s.add(line.rstrip("\r\n"))

for item in s:
  print(item)
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



### Replace all instances of `"l"` in string `"hello"`

```python
"hello".replace("l", "x")
```

```clojure
(require '[clojure.string :as str])
(str/replace "hello" "l" "x")
```



### Replace first instance of `"l"` in string `"hello"`

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
#= ['hello', '', 'world', 'test', '', '']

"hello  world test  ".split()   
#= ['hello', 'world', 'test']
```

```clojure
(require '[clojure.string :as str])

(str/split "hello  world test  " #" ")
;= ["hello" "" "world" "test"]

(str/split "hello  world test  " #" +")
;= ["hello" "world" "test"]
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
new_record
```

```clojure
(let [record     {:name {:first "Kim", :last "Schmitz"} :age 31 :secret "delete-me"}
      new-record (-> record
                     (assoc-in [:name :last] "Dotcom")
                     (dissoc :secret)
                     (update-in [:age] inc))]
  new-record)
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



### Get the sum of each row in a matrix

```python
list(map(sum, [[1, 2, 3], [3, 4, 5]]))
```

```clojure
(mapv (partial reduce +) [[1 2 3] [3 4 5]])
```



### Get the sum of each column in a matrix

```python
list(map(sum, zip(*[[1, 1], [2, 2], [3, 5]])))
```

```clojure
(mapv (partial reduce +) (apply map list [[1 1] [2 2] [3 5]]))
```

This works because Clojure's `map` passes one argument for every sequence into the fn; try `(map vector [1 2] [3 4])` or `(map + [1 2] [3 4])`



### Get milliseconds since epoch

```python
import time

time.time() * 1000
# returns a float
```

```clojure
(System/currentTimeMillis)
; returns a Long
```


### Get an ISO 8601 representation of the date and time

```python
import pytz
from datetime import datetime

datetime.now(pytz.utc).isoformat()
#= '2014-03-27T01:20:15.065206+00:00'
```

(Requires [pytz](https://pypi.python.org/pypi/pytz); `sudo apt-get install python-tz python3-tz`.)

Or, if you absolutely cannot install pytz:

```python
from datetime import datetime

datetime.utcnow().isoformat() + "Z"
#= '2014-03-27T01:20:15.065206Z'
```

```clojure
(.toString (java.time.Instant/now))
;= "2014-03-27T01:32:13.032Z"
```

(Requires Java 8+.)



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



### Convert a unicode string to bytes

```py
u"hello \ucccc".encode("utf-8")
```

```clj
(.getBytes "hello \ucccc" "UTF-8")
```



### Convert bytes to a unicode string

```py
b"hello \xec\xb3\x8c".decode("utf-8")
```

```clj
(String. (.getBytes "hello \ucccc" "UTF-8") "UTF-8")
```

(Clojure has no bytes literal, so we call `.getBytes` to make a `byte[]` first.)



### Get a SHA1 digest or hex digest for a string

```python
import hashlib

hashlib.sha1(b"hello world").digest()
hashlib.sha1(b"hello world").hexdigest()

hashlib.sha1(u"hello world \ucccc".encode("utf-8")).digest()
hashlib.sha1(u"hello world \ucccc".encode("utf-8")).hexdigest()
```

```clojure
(import '[java.security MessageDigest])

(defn hexify [digest]
  (apply str (map #(format "%02x" (bit-and % 0xff)) digest)))

(defn sha1digest [b]
  (.digest (java.security.MessageDigest/getInstance "sha1") b))

(sha1digest (.getBytes "hello world" "UTF-8"))
(hexify (sha1digest (.getBytes "hello world" "UTF-8")))

(sha1digest (.getBytes "hello world \ucccc" "UTF-8"))
(hexify (sha1digest (.getBytes "hello world \ucccc" "UTF-8")))
```



### Get a SHA1 hex digest for a large file

```python
# Traditional implementation with string objects

from __future__ import print_function
import hashlib

def sha1_for_file(f, block_size=2**20):
  assert block_size >= 1, "block_size must be >= 1, was %r" % (block_size,)
  sha1 = hashlib.sha1()
  while True:
    data = f.read(block_size)
    if not data:
      break
    sha1.update(data)
  return sha1

with open("/etc/passwd", "rb") as f:
  print(sha1_for_file(f).hexdigest())
```

```python
# bytearray implementation that avoids creating string objects, to avoid GC pressure on PyPy/Jython

from __future__ import print_function
import hashlib

def sha1_for_file(f, block_size=2**20):
  assert block_size >= 1, "block_size must be >= 1, was %r" % (block_size,)
  sha1 = hashlib.sha1()
  arr = bytearray(block_size)
  while True:
    num_bytes_read = f.readinto(arr)
    if not num_bytes_read:
      break
    if num_bytes_read < block_size:
      sha1.update(arr[:num_bytes_read])
    else:
      sha1.update(arr)
  return sha1

with open("/etc/passwd", "rb") as f:
  print(sha1_for_file(f).hexdigest())
```

```clojure
(require '[clojure.java.io :refer [as-file input-stream]])
(import '[java.security MessageDigest])

(defn hexify [digest]
  (apply str (map #(format "%02x" (bit-and % 0xff)) digest)))

(defn sha1-for-file
  ([file]
    (sha1-for-file file 1048576))
  ([file block-size]
    (assert (pos? block-size) (str "block-size must be >= 1, was " block-size))
    (with-open [fis (input-stream file)]
      (let [arr (byte-array block-size)
            digest (java.security.MessageDigest/getInstance "sha1")]
        (loop []
          (let [num-bytes-read (.read fis arr 0 block-size)]
            (when (not= num-bytes-read -1)
              (.update digest arr 0 num-bytes-read)
              (recur))))
        digest))))

(hexify (.digest (sha1-for-file (as-file "/etc/passwd"))))
```



### Print names of everything in a module (Python) or namespace (Clojure)

```python
import os
dir(os)
```

```clojure
(keys (ns-map 'clojure.core))
```



### Get the value of the last evaluation in the REPL

Python:

```python
_
```

Clojure:

```clojure
*1
```

(Clojure also supports `*2` and `*3` to get even older evaluations.)



## Other Clojure tips


### Get a traceback for the last exception thrown in the REPL

```clojure
(pst)
```

(<b>p</b>rint <b>s</b>tack <b>t</b>race)



### Clojure says "IllegalArgumentException: No matching method" but the method exists!

Unlike Python, a Java class can have multiple methods of the same name, as long as the method signature is different.  For each call site, Java selects the correct method at compile time.

If you see this error, you may have passed in the wrong number of arguments to the method.



### I've passed in the right number of arguments but it still says "IllegalArgumentException No matching method", or I get a strange ClassCastException.

This can happen if you're calling a Java varargs method.  Varargs methods actually take an array, not a variable number of arguments.  The Java compiler automatically creates these argument arrays for you, but Clojure does not.

For example, in Clojure, instead of the **incorrect**

```clojure
(.getPath (java.nio.file.FileSystems/getDefault) "/etc" "passwd")
```

you must call

```clojure
(.getPath (java.nio.file.FileSystems/getDefault) "/etc" (into-array String ["passwd"]))
```

You must pass an array even if you call with 0 varargs.  See [CLJ-440](http://dev.clojure.org/jira/browse/CLJ-440).



### Call a private function

```clojure
(#'other.namespace/fun)
```

(Get the var and call it, which calls the function.)



### Evaluate a private object

```clojure
@#'other.namespace/obj
```

(Get the var and deref it.)



### Comment out the next form

```clojure
[1 2 #_ 3 4 5]
;= [1 2 4 5]
```



### Comment out the next two forms

```clojure
{"a" 1 #_ #_ "b" 2 "c" 3}
;= {"a" 1, "c" 3}
```



## Leiningen tips


### Set up `(r)` to reload code and `(t)` to run tests in REPL

This sets up an `(r)` that calls `tools.namespace`'s `(refresh)` but also [reloads aliased namespaces](https://groups.google.com/forum/#!msg/clojure/dsfmYU4YMSM/Gq3uo9tI748J), and a `(t)` that calls `(r)` and runs your tests.

Add a `tools.namespace` dependency and a `:repl-options :init` to your `project.clj`:

```clojure
(defproject myproject "0.1.0-SNAPSHOT"
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [org.clojure/tools.namespace "0.2.4"]]
  :repl-options {:init (do
                         (require '[myproject.core :as m]) ; convenience
                         (require '[clojure.tools.namespace.repl :refer [refresh]])
                         (require '[clojure.test])

                         (defn r
                           "Refresh code and fix up aliases in the user namespace"
                           []
                           (let [result (refresh)]
                             ; https://github.com/clojure/tools.namespace#warnings-for-aliases
                             (doseq [[sym target-ns] (ns-aliases 'user)]
                               (ns-unalias 'user sym)
                               ; (the-ns (ns-name ...)) to get the post-refresh namespace object
                               (.addAlias (the-ns 'user) sym (the-ns (ns-name target-ns))))
                             result))

                         (defn t
                           "Refresh code and run tests"
                           []
                           (let [result (r)]
                             (if (not= result :ok)
                               result
                               (clojure.test/test-ns 'myproject.core-test)))))
                })
```

Adjust the `(require '[myproject.core :as m])` and `(clojure.test/test-ns ...)` lines to the correct namespace.

Then, `lein repl` and `(r)` or `(t)` in your REPL.

See the [tools.namespace README](https://github.com/clojure/tools.namespace) for information on how it works.

Running code from a network drive and `(refresh)` or `(r)` or `(t)` not loading updated code?  Make sure the local and remote machine clocks are in sync.  `tools.namespace` uses

```clojure
(defn- modified-files [tracker files]
  (filter #(< (::time tracker 0) (.lastModified ^File %)) files))
```



### Make `lein repl`, `lein run`, and other project commands run at full JVM performance; not optimized for startup time

Add this to your `project.clj`:

```clojure
:jvm-opts ^:replace ["-server"]
```



### Make `lein repl`, `lein run`, and other project commands run at full JVM performance, and give JVM 2GB of heap

Add this to your `project.clj`:

```clojure
:jvm-opts ^:replace ["-server" "-Xmx2G"]
```



### Make `lein repl` not slow when the project is on a network drive

Add this to your `project.clj`:

```clojure
:repl-options {:history-file nil}
```



### Start a REPL without creating a local security hole

`lein repl` starts an nREPL server that anyone on your machine can connect to, including untrusted software running in other user accounts.  This is a problem even on single-human-user machines.

nREPL does not provide any way to require authorization, but you can avoid using nREPL and just use the underlying REPL library that `lein repl` uses: [REPL-y](https://github.com/trptcolin/reply).

First, add `[reply "0.3.5"]` your `:dependencies` and move your `:repl-options :init` code to `init.clj`:

```clojure
(defproject myproject "0.1.0-SNAPSHOT"
  :source-paths ["src" "test"]
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [org.clojure/tools.namespace "0.2.4"]
                 [reply "0.3.5"]]
  :repl-options {; Must use (do ...) block to avoid breaking with plugins
                 :init (do (load-file "init.clj"))
                 })
```

Then, create an executable script called `repl` with these contents:

```bash
#!/bin/bash

set -e

cd "$(dirname "${BASH_SOURCE[0]}")"
# --standalone is **required** to avoid starting an nREPL listener
java -Dfile.encoding=UTF-8 -cp "$(lein classpath)" reply.ReplyMain --standalone -e '(load-file "init.clj")'
```

`./repl` will start a REPL *without* starting an nREPL listener.

If you want to save a few seconds per REPL launch, you can use a more complicated `repl` that uses a cached classpath when `project.clj` is unchanged:

```bash
#!/bin/bash

set -e

cd "$(dirname "${BASH_SOURCE[0]}")"

mkdir -p .lein-cp-cache
# md5sum on Linux, md5 on OS X
PROJECT_CLJ_CHECKSUM="$(cat project.clj | (md5sum 2> /dev/null || md5) | cut -f 1 -d " ")"
# Use the cached classpath if one exists, else calculate the classpath
# and cache it.  If it's already cached, we save ~2 seconds.
if [ -f ".lein-cp-cache/$PROJECT_CLJ_CHECKSUM" ]; then
  REPLY_CP="$(cat ".lein-cp-cache/$PROJECT_CLJ_CHECKSUM")"
else
  REPLY_CP="$(lein classpath)"
  echo "$REPLY_CP" > ".lein-cp-cache/$PROJECT_CLJ_CHECKSUM"
fi

# --standalone is **required** to avoid starting an nREPL listener
java -Dfile.encoding=UTF-8 -cp "$REPLY_CP" reply.ReplyMain --standalone -e '(load-file "init.clj")'
```

Other approaches to solving this problem (though they did not work well for me) include [nreplds](https://github.com/monsanto/nreplds), [ssh-repl](https://github.com/mtnygard/ssh-repl), and using `iptables -A OUTPUT -m owner` to reject local outbound connections.



## ClojureScript tips


### Get a ClojureScript REPL powered by phantomjs

```
sudo apt-get install phantomjs
```

Make sure you have ClojureScript in the `:dependencies` vector in your `project.clj`; e.g. `[org.clojure/clojurescript "0.0-2202"]` ([the latest version is listed here](http://search.maven.org/#search|ga|1|org.clojure/clojurescript)).

Add [austin](https://github.com/cemerick/austin) to the `:plugins` vector in your `project.clj`, adding a `:plugins` key if necessary:

```clojure
:plugins [[com.cemerick/austin "0.1.4"]]
```

(Don't add it to your `~/.lein/profiles.clj`, because you don't want to slow down startup for all projects.)

Start a Clojure REPL as usual with `lein repl`, then run

```clojure
(cemerick.austin.repls/exec)
```



### Get a ClojureScript REPL powered by your browser

Make sure you have ClojureScript in the `:dependencies` vector in your `project.clj`; e.g. `[org.clojure/clojurescript "0.0-2202"]` ([the latest version is listed here](http://search.maven.org/#search|ga|1|org.clojure/clojurescript)).

Add [austin](https://github.com/cemerick/austin) to the `:plugins` vector in your `project.clj`, adding a `:plugins` key if necessary:

```clojure
:plugins [[com.cemerick/austin "0.1.4"]]
```

(Don't add it to your `~/.lein/profiles.clj`, because you don't want to slow down startup for all projects.)

Start a Clojure REPL as usual with `lein repl`, then run

```clojure
(cemerick.piggieback/cljs-repl :repl-env (cemerick.austin/repl-env))
```

The output will include a `Browser-REPL ready @ URL` line.  Open that URL in your browser.  The page will be blank, as it serves only as an engine for Austin.  Switch back to your terminal and enter ClojureScript code there.



## Other documentation

[The Joy of Clojure](http://joyofclojure.com/) is an excellent book that will teach you to write idiomatic Clojure.

[clojure-doc.org](http://clojure-doc.org/) covers the language essentials.

[Rich Hickey's talks](http://thechangelog.com/rich-hickeys-greatest-hits/), [more videos on this list](http://alexott.net/en/clojure/video.html), and [even more on YouTube](https://www.youtube.com/user/ClojureTV/videos?flow=list&live_view=500&sort=dd&view=0).

clojure.org has a [cheatsheet](http://clojure.org/cheatsheet) covering core functions and [detailed implementation information](http://clojure.org/documentation).
