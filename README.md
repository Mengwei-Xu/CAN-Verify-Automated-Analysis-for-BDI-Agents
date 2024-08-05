# CAN-Verify

---

## Description

CAN-Verify automates the process of formal modelling BDI agents in the CAN language.

It handles the program encoding to Bigraphs, model execution in BigraphER, and model-checking with with PRISM. It was developed by Thibault Rivoalen
([thibault.rivoalen@alumni.enac.fr](mailto:thibault.rivoalen@alumni.enac.fr)) for the University of Glasgow, UK. If you require assistance, please email:
please also send the email to corresponding author to [mengwei.xu@newcastle.ac.uk](mailto:mengwei.xu@newcastle.ac.uk)

---

## Quick Paper Reproduction Script

Can-Verify has been tested on Ubuntu linux, and should work on other linux like systems.

The test script ```install-and-run.sh``` will try to 1. automatically install the dependencies and 2. run the IFM2023 accepted paper 
examples ("CAN-verify: Verification Tool for BDI Agents, Mengwei Xu, Thibault Rivoalen, Blair Archibald, and Michele Sevegnani, In 18th International Conference on integrated Formal Methods (iFM), pages 364-373, 2023") and one extra travelling example.

You may need to do ```chmod +x install-and-run.sh``` first to ensure the correct execute bit is set.

---
## Quick Start

We have pre-compiled the main binary and dependencies in ```./bins```.

1. ./CAN-Verify is the main CAN-Verify program
2. ./bigrapher is the binary for dependency tool BigraphER

To build the PRISM model checker, we have provided the source code (for Linux x86) downloaded directly from the PRISM website.

- extract the prism source code
- navigate to the folder as current path
- simply run ```./install.sh```

To allow the binary dependencies to be discovered, please set these in your PATH:

```export PATH=./bins:./bins/prism-4.8-linux64-x86/bin:$PATH```

You may also need to run ```chmod u+x bigrapher``` to set bigrapher as executable

If these pre-compiled binaries do not work, you may need to build from source as below.

---

## Building from Source

CAN-Verify has the following dependencies:

1. Java 17 (or above),
2. PRISM model checker: http://www.prismmodelchecker.org/download.php
3. BigraphER: https://uog-bigraph.bitbucket.io/
4. Opam/OCaml: ```sudo apt install ocaml opam```
5. OCaml Dependencies: dune, menhir, odoc. Packages can be installed using opam with a command such as: ```opam install dune menhir odoc```

PRISM and bigrapher must be available on your path for CAN-Verify to work correctly.

- We have successfully tested that the current tool works with BigraphER 1.9.3, PRISM 4.8.1, Opam 2.1.5, and Ocaml 4.14.1. 

### Building CAN-Verify

The given Makefile can be used to CAN-Verify the program via ```make```.

## Building with Docker

Make sure the current path is in the extracted zip fold.

Run ```sudo make docker``` to have a ```can-verify``` Docker image built for you.



---

## Usage


Run ```./CAN-Verify  [options] [-p prop.txt] <file.can>```

### Options: ``` [options] ```

```-static```: performs a static check of ```file.can``` for common errors, e.g. syntax issues, no plan to address an event etc.

```-dynamic```: performs dynamic model checking by symbolically creating all executions in BigraphER and checking with PRISM. Use this alongside a property file `-p` to determine if your properties hold 

```-p```: sets the program which file contains the properties to verify 

```-Ms```: sets the maximum number of states possible (default: 4000) 

```-mp```:  sets the minimum number of plans required in static checks (default: 2) 

```-Mp```:  sets the maximum number of plan allowed in stacic checks (default: 100) 

```-mc```:  sets the minimum number of character required in a name (default: 2) 

```-Mc```:  sets the maximum number of character allowed in a name (default: 20)

```-big```:  Exports the CAN model to the BigraphER equivalent (.big) and exits

```--help```:  to display this list of options

### Belief-based property specifications: ``` [-p prop.txt] ```

Properties are specified in structured natural language, where the "belief" is a user defined belief.

You should write (in ```prop.txt```) one of the:

1. In all possible executions, eventually the belief ```variable``` holds: *A [ F ("```variable```") ]*.
2. In some executions, eventually the belief ```variable``` holds: *E [ F ("```variable```") ]*.

For example, we may have:

1. In all possible executions, eventually the belief F1_clean holds.
2. In some executions, eventually the belief F1_clean holds.

A Parser error will be given if the exact wording is not followed.

### Generic properties
Generic properties are always included by default and check if for some/all executions an event finishes with failure or success.

### Docker Usage

Instead of running ```./CAN-Verify  [options] [-p prop.txt] <file.can>```, in docker setting, please run

```sudo docker run --rm --volume "${PWD}":/test_rep --interactive can-verify [options] [-p prop.txt] <file.can>```.

For example to run the example in listing 1.4, you can run

```sudo docker run --rm --volume "${PWD}":/test_rep --interactive can-verify -dynamic -p paper_examples/Listing_1-4.txt paper_examples/Listing_1-4.can```


## Paper Examples
As per our accepted IFM paper ("CAN-verify: Verification Tool for BDI Agents, Mengwei Xu, Thibault Rivoalen, Blair Archibald, and Michele Sevegnani, In 18th International Conference on integrated Formal Methods (iFM), pages 364-373, 2023") attached for the artifact submission, the examples used in the paper are included in the folder ./paper_examples. The following is the commentary:

- **Listing_1-3.can** corresponds to the examples in Listing 1.3. CAN agent for concurrent sensing in UAVs.
- **Listing_1-3-Corrected.can** corresponds to the improved design for Listing 1.3 where we replace the concurrenty program  **dust || photo** with **dust; photo**.
- **Listing_1-4.can** corresponds to the examples in Listing 1.4. CAN agent for two-storey building patrol robot.
- **Listing_1-4.txt** corresponds to the belief-based property spefication for Listing 1.4.


There is an extra example Listing_1.can, which is also included in the folder ./paper_examples, to describe the simple agent in the travelling scenario from the home to the work. 

The Listing_1.txt corresponds to the belief-based property spefication for Listing 1.can, checking that the agent can always believe that it will eventually be at work. 


### Running the Examples
For the example in listing 1.3, please run the command

```./CAN-Verify -dynamic paper_examples/Listing_1-3.can```

```./CAN-Verify -dynamic paper_examples/Listing_1-3-Corrected.can```

For the example in listing 1.4, please run the command


```./CAN-Verify -dynamic -p paper_examples/Listing_1-4.txt paper_examples/Listing_1-4.can```


#### Expected Outputs

For the example in listing 1.3, you should get the following

> Model checking: A [ F ("no_failure"&(X "empty_intention")) ] ... Result: false

This means that it is not always the case the task of sensing is achieved eventually.

> Model checking: E [ F ("failure"&(X "empty_intention")) ] ... Result: true

This means that there indeed exists a case that the task of sensing is failed eventually.

These two outputs are important as they confirm that there is a possibility that a bad interleaving may cause some troubles. 

If you run the exmaple in **Listing_1-3-Corrected.can**, you should get the following

> Model checking: A [ F ("no_failure"&(X "empty_intention")) ] ... Result: True

This means that it is always the case the task of sensing is achieved eventually.

> Model checking: E [ F ("failure"&(X "empty_intention")) ] ... Result: false

This means that there never exists the case the task of sensing is failed eventually.

For the exmaple in **Listing_1-4.can**, you should get the following

> Model checking: A [ F ("predicate_F1_clean") ] ... Result: true

This means that it is always the case that the predicate of F1_clean holds eventually.


If you run the exmaple in **Listing_1.can**, you should get the following

> Model checking: A [ F ("predicate_at_work") ] ... Result: true

This means that it is always the case that the predicate of at_work holds eventually.

This example is designed correctly to have a full coverage of the plans to ensure the agent can always travel to the work successfully.
The output of the result above simply confirms our correct design. 


## Source Code Overview

CAN-Verify consists of three main components

- `CAN2BigraphERCompiler`: converts CAN formatted agent programs into their bigraph equivalent (following the scheme of "Modelling and Verifying BDI Agents
with Bigraphs Blair Archibald, Muffy Calder, Michele Sevegnani, and Mengwei Xu Science of Computer Programming, Volume 215, pp 288-310, Elsevier, March 2022"). The accepted syntax of CAN programs is given in the same paper.

- `PropertiesCompiler`: Handles the conversion of natural language-like properties (given above) into their CTL properties for use in PRISM

- `CANVerifier`: Handles the argument passing and calls `CAN2BigraphERCompiler` and `PropertiesCompiler` as required to construct a bigraph model, executes it using BigraphER, and performs model checking (if requested) via PRISM. This interaction with bigraphER and PRISM is handled directly through system calls to allow flexibility, i.e. other model checkers can be added in future.

## Known Issues

- Preexisting binaries may not work on older versions of Ubuntu (<= 20.04).

- Preexisting binaries are untested on other linux variants.


## Copyright and license

Copyright 2012-2022 Glasgow Bigraph Team 

All rights reserved. Tools distributed under the terms of the Simplified BSD License that can be found in the [LICENSE file](LICENSE.md)