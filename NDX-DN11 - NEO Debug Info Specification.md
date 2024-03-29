<!-- markdownlint-enable -->
# NDX-DN11 - NEO Debug Info Specification

- Author: Harry Pierson (harrypierson@ngd.neo.org)
- Status: Draft

## Abstract

This design note describes the debug information format used by
the [NEO Smart Contract Debugger](NDX-DN04%20-%20NEO%20Smart%20Contract%20Debugging.md).
This information will be generated by smart contract compilers
such as [NEON](https://github.com/neo-project/neo-devpack-dotnet)
or [neo-boa](https://github.com/CityOfZion/neo-boa).

## Current Status

The current version of the debugger uses a temporary debug info
format generated by a
[fork of the NEON compiler](https://github.com/neo-project/neo-devpack-dotnet/tree/dehvawk/neon-de).
This format was has not been peer reviewed or optimized for size or performance.
There is no expectation that the current format will be the final format.
At a minimum, the format needs to be improved to eliminate redundant information
that bloats the file size. Realistically, representatives from other smart
contract compiler projects needs to be given an opportunity to provide input
so that the final format can be used across the various languages in the NEO
ecosystem.

## Current Format

The NEON-DE compiler emits debug info in a json file with the extension
.debug.json. This document contains the following types:

> Note, these types are defined in TypeScript for readability.
> There is no requirement that this format be implemented in TypeScript.

``` typescript
interface DebugInformatiom {
    entrypoint: string;
    methods: Method[];
    sequence-points: SequencePoint[]; // unused, to be removed
}

interface Method {
    name: string;
    namespace: string;
    display-name: string;
    start-address: number;
    end-address: number;
    parameters: Variable[];
    return-type: string;
    variables: Variable[];
    sequence-points: SequencePoint[];
}

interface Variable {
    name: string;
    type: string;
}

interface SequencePoint {
    address: number;
    document: string;
    start-line: number;
    start-column: number;
    end-line: number;
    end-column: number;
}
```

A debug info document is primarily an array of Method objects. Each opcode
in a compiled NeoVM script is associated with exactly one Method object in
the debug info. Each Method object has a start and end address than cannot
overlap with any other Method object in the debug info. The Method for a
given address is determined by searching the non-overlapping list of method
address ranges for the single Method the opcode address belongs to.

In addition to the address range, a Method object contains the following
information:

- Display information about the Method, such as `name`, `namespace` and
  `display-name`. The `display-name` field is used to identify the Method
  in the call stack view of the debugger.
- Type information about parameters, variables and return variables.
  The type information is used to format the memory associated with type
  for human consumption in the debugger. Parameters and variables also
  have a name that is displayed in the debugger, allowing the developer
  to associate individual types in the variable window to variables
  in their code.
- Sequence points map individual NeoVM opcode addresses to a sequence
  of text in a source file. Because of the nature of mapping high level
  source code to low level opcodes, there will not be a sequence point
  for every address in the NeoVM script file. 

> Note, the `sequence-points` property of the root debug info object
> is not used by the NEO smart contract debugger and will be removed
> from the NEON compiler in a future release.
