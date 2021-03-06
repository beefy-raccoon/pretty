Columnar Output
===============

The ``io.aviso.columns`` namespace is what's used by the exceptions namespace to format the exceptions, properties, and stack traces.

The ``format-columns`` function is provided with a number of column definitions, each of which describes the width and justification of a column. 
Some column definitions are just a string to be written for that column, such as a column separator.
``format-columns`` returns a function that accepts a StringWriter (such as ``*out*``) and the column values.

``write-rows`` takes the function provided by ``format-columns``, plus a set of functions to extract column values,
plus a seq of rows.
In most cases, the rows are maps, and the extraction functions are keywords (isn't Clojure magical that way?).

Here's an example, from the exception namespace:

.. code-block:: clojure

  (defn- write-stack-trace
    [writer exception]
    (let [elements (->> exception expand-stack-trace (map preformat-stack-frame))
          formatter (c/format-columns [:right (c/max-value-length elements :formatted-name)]
                                      "  " (:source *fonts*)
                                      [:right (c/max-value-length elements :file)]
                                      2
                                      [:right (->> elements (map :line) (map str) c/max-length)]
                                      (:reset *fonts*))]
      (c/write-rows writer formatter [:formatted-name
                                      :file
                                      #(if (:line %) ": ")
                                      :line]
                    elements)))