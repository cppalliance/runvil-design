Runvil runtime core.

This crate holds the [`Run`] state machine that turns a parsed prompt into a run; the parser that reads a prompt markdown file into a [`Prompt`](runvil_parser::Prompt) is `runvil-parser`'s. A run executes H1 once before walking sections top to bottom (fall-through), issuing every model round, tool call, store operation, and timer as an [`Effect`] value (`Chat`, `ToolCall`, `Vfs`, `Timer`) the Harness performs and answers, and reporting every boundary as an [`Event`](runvil_types::event::Event) value the Harness records. The Engine emits an effect whenever a run needs something from outside and takes its clock from the run's context; the Harness steps each run through the `runvil` facade, performs every effect, and records each run through the Host's recorder. The public vocabulary a run is configured with - the model and tool catalogs, the tool contract, the event enum - sits in the `runvil-types` crate; the chat vocabulary a `Chat` effect holds and its answer returns is `runvil-model-client`'s. The store handle the Harness seeds or extracts comes from `runvil-vfs`. No model client is defined here: the Harness owns the transport that performs a round and reaches the vocabulary through the facade.

A source is a runvil prompt only when its frontmatter declares a `runvil:` version; [`runvil_version`](runvil_parser::runvil_version) reports it (or `None`), and the runtime refuses a source that lacks a supported version.

# Examples

Detect a runvil source and parse it into a [`Prompt`](runvil_parser::Prompt):

```
use runvil::Prompt;
use runvil_parser::runvil_version;

let source = "---\nname: greeter\ndescription: says hi\nrunvil: 0\n---\n\n# Greeter\n\n## Say hi\n\nSay hello.\n\n```lua\nreturn models.infer(prose)\n```\n";

// Version detection gates whether the runtime will accept the source.
assert_eq!(runvil_version(source), Some(0));
assert_eq!(runvil_version("plain text, no frontmatter"), None);

// A parse returns its parse-time events beside the outcome.
let (prompt, events) = Prompt::parse(source, "doc-example");
let prompt = prompt?;
assert!(!events.is_empty());
assert_eq!(prompt.title(), "Greeter");
assert_eq!(prompt.frontmatter().name(), "greeter");
# Ok::<(), runvil::ParseError>(())
```

Executing a parsed prompt builds a [`Run`] over a [`RunContext`] prepared by an [`Environment`] (which holds the catalog of tools the Harness activated); the filesystem handle - real directories and the declared store - sits on the context, defaulting to a fresh in-memory store at `/`. The Harness then loops: [`Run::step`] returns the effects to perform and the events to log, and [`Run::resume`] hands each effect's answer back.
