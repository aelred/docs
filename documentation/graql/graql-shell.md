# Graql Shell

The Graql shell is used to execute Graql queries from the command line, or to
let Graql be invoked from other applications.

The Graql shell requires Mindmaps to already be running in the background, using `./mindmaps.sh start`.

## Interactive Shell

The Graql shell is contained in the `bin` folder.

If the shell is executed without any parameters, it will open a REPL (Read
Evaluate Print Loop):

```bash
bin/graql.sh
>>> match $x id "Pikachu";
$x id "Pikachu" isa pokemon;
```

The user can enter a query and press enter for it to be evaluated. `match`
queries will return all results, `ask` queries will return `true`
or `false`. `insert` and `delete` queries will execute with no output.

The interactive shell will never commit to the graph unless the user types
`commit`.

The REPL features several special commands:
- `commit` - Commits and validates the graph. If validation fails, the graph
  will not commit.
- `edit` - Opens the user's default text editor, specified by the `$EDITOR`
  environment variable. By default set to `vim`. When the editor exits, the
  query is executed in the shell.
- `load <file>` - Load the given Graql file.

## Arguments

The Graql shell accepts several arguments:

| Long Option | Option | Description                                              |
| ----------- | ------ | -------------------------------------------------------- |
| --name      | -n     | Name of the graph to load.                               |
| --uri       | -u     | The URI of the engine instance to connect to.            |
| --execute   | -e     | A query to execute.                                      |
| --file      | -f     | A path to a file containg a query to execute and commit. |

If either `--execute` or `--file` is provided, the REPL will not open and the
graph will be automatically committed.
