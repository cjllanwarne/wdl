Workflow Description Language (wdl)
===================================

The Workflow Description Language is a language for describing dependency trees of tasks (algorithms) in a concise and clear syntax.

Installation
============

Generating the JAR
------------------

To build the JAR file, run:

```
$ mvn package
```

Which will create a file target/Wdl-${version}.jar as an executable JAR.  To invoke the CLI:

```
$ java -jar target/Wdl-0.0.4.jar examples/0.wdl ast
```

Generating the parser code
--------------------------

Use the Hermes parser generator (http://github.com/scottfrazer/hermes).  From the root of this repository, run:

```
$ hermes generate grammars/composite_task.zgr --directory=src/main/java/org/broadinstitute/compositetask --language=java --java-package=org.broadinstitute.compositetask
```

Or, run the Ant target `generate-parser` which will run the command above:

```
$ ant generate-parser
```

Usage
=====

Java API
--------

From Java code, the main interface is the CompositeTask, which can be used in this example to print out the immediate children nodes of this composite task:

```java
CompositeTask ct = new CompositeTask(new File(args[0]));
for ( CompositeTaskNode entry : ct.getNodes() ) {
  System.out.println("Node: " + entry);
}
```

Command-line Interface
----------------------

The command line interface provides some common tools for analyzing and displaying WDL files Below are the actions that can be taken by running the executable JAR without any parameters:

```
$ java -jar target/Wdl-0.0.4.jar
Usage: <.wdl file> <tokens,astarsetree,entities,graph,format,format-ansi,format-html,replace>

Actions:
  tokens: tokenize the source code
  ast: parse source code and output an abstract syntax tree
  parsetree: parse source code and output a parsetree
  entities: output an abbreviated view of all entities and which scope they're nested in
  graph: output the set of verticies and edges for the directed acyclic graph
  format: reformat source code
  format-ansi: reformat source code and colorize for the terminal
  format-html: reformat source code and add HTML span tags
  replace <task[:version]> <new task:version>: replace a task/version with a different task/version
```

CLI Examples
------------

The WDL file we'll use is:

```
$ cat examples/7.wdl
composite_task test {
  step s0[version=0] {
    output: File("abc") as foo;
  }

  for (I in L) {
    for (J in M) {
      step s1[version=0] {
        input: p0=I, p1=J, p2=foo;
        output: File("def") as bar;
      }
    }
  }

  step s2[version=0] {
    input: p0=bar;
  }
}
```

Get the abstract syntax tree:

```
$ java -jar dist/Wdl-0.0.4.jar examples/7.wdl ast
(CompositeTask:
  body=[
    (Step:
      body=[
        (StepOutputList:
          outputs=[
            (StepFileOutput:
              as=(Variable:
                member=None,
                name=identifier
              ),
              file=string
            )
          ]
        )
      ],
      task=(Task:
        attributes=[
          (TaskAttribute:
            value=number,
            key=identifier
          )
        ],
        name=identifier
      ),
      name=None
    ),
    (ForLoop:
      body=[
        (ForLoop:
          body=[
            (Step:
              body=[
                (StepInputList:
                  inputs=[
                    (StepInput:
                      parameter=identifier,
                      value=(Variable:
                        member=None,
                        name=identifier
                      )
                    ),
                    (StepInput:
                      parameter=identifier,
                      value=(Variable:
                        member=None,
                        name=identifier
                      )
                    ),
                    (StepInput:
                      parameter=identifier,
                      value=(Variable:
                        member=None,
                        name=identifier
                      )
                    )
                  ]
                ),
                (StepOutputList:
                  outputs=[
                    (StepFileOutput:
                      as=(Variable:
                        member=None,
                        name=identifier
                      ),
                      file=string
                    )
                  ]
                )
              ],
              task=(Task:
                attributes=[
                  (TaskAttribute:
                    value=number,
                    key=identifier
                  )
                ],
                name=identifier
              ),
              name=None
            )
          ],
          item=identifier,
          collection=identifier
        )
      ],
      item=identifier,
      collection=identifier
    ),
    (Step:
      body=[
        (StepInputList:
          inputs=[
            (StepInput:
              parameter=identifier,
              value=(Variable:
                member=None,
                name=identifier
              )
            )
          ]
        )
      ],
      task=(Task:
        attributes=[
          (TaskAttribute:
            value=number,
            key=identifier
          )
        ],
        name=identifier
      ),
      name=None
    )
  ],
  name=identifier
)
```

Get a view of the graph

```
$ java -jar dist/Wdl-0.0.4.jar examples/7.wdl graph
VERTICIES
---------
[Step: name=s1]
[Variable: name=J]
[Variable: name=M]
[Variable: name=I]
[Variable: name=L]
[Variable: name=foo]
[Step: name=s0]
[Variable: name=bar]
[Step: name=s2]
[CompositeTaskForScope: collection=[Variable: name=L], var=[Variable: name=I], # nodes=1]
[CompositeTaskForScope: collection=[Variable: name=M], var=[Variable: name=J], # nodes=1]

EDGES
-----
[Edge
  from: [Step: name=s1]
  to: [Variable: name=bar]
]
[Edge
  from: [Variable: name=L]
  to: [CompositeTaskForScope: collection=[Variable: name=L], var=[Variable: name=I], # nodes=1]
]
[Edge
  from: [Variable: name=bar]
  to: [Step: name=s2]
]
[Edge
  from: [Variable: name=foo]
  to: [Step: name=s1]
]
[Edge
  from: [Step: name=s0]
  to: [Variable: name=foo]
]
[Edge
  from: [CompositeTaskForScope: collection=[Variable: name=M], var=[Variable: name=J], # nodes=1]
  to: [Variable: name=J]
]
[Edge
  from: [Variable: name=J]
  to: [Step: name=s1]
]
[Edge
  from: [Variable: name=M]
  to: [CompositeTaskForScope: collection=[Variable: name=M], var=[Variable: name=J], # nodes=1]
]
[Edge
  from: [CompositeTaskForScope: collection=[Variable: name=L], var=[Variable: name=I], # nodes=1]
  to: [Step: name=s2]
]
[Edge
  from: [Variable: name=I]
  to: [Step: name=s1]
]
[Edge
  from: [CompositeTaskForScope: collection=[Variable: name=L], var=[Variable: name=I], # nodes=1]
  to: [Variable: name=I]
]
```
