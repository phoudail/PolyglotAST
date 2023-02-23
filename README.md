# DEPRECATED

Note: due to JSitter not being maintained and up to date with the tree-sitter grammars and API, this repositery is currently not suitable for development. It might be picked up again in the future, but until then consider using the Rust implementation instead, available at https://github.com/phoudail/polyglot_ast

This repository contains tools to build polyglot ASTs from source code of GraalVM applications. It currently partially supports python and javascript, and is built in a manner that aims to facilitate addition of new languages.  
You will need JSitter in order to be able to run the project, which is available at https://github.com/JetBrains/jsitter  

There are two main classes to be looking at; they are introduced after the _getting started_ below.

# Getting started

Prerequisites: 
- Highest recommendation to run on Linux. MacOS should be possible, but you are on your own to resolve errors.
- You will need a JVM, Maven, CMake and Clang installed on your system
- While not necessary to extend the framework, the examples provided in the GraalSamples folder will require an installation of GraalVM

1. For jsitter, clone the jsitter repo: https://github.com/quentinLeDilavrec/jsitter
2. Fix the merge conflict in jsitter/native/tree_sitter/lib/include/tree_sitter/api.h around line 670 (take the HEAD)
3. In jsitter/native, edit the file CMakeLists.txt to add the python and javascript libraries: 
```cmake
add_library(tspython SHARED grammars/tree-sitter-python/src/parser.c grammars/tree-sitter-python/src/scanner.cc)
add_library(tsjavascript SHARED grammars/tree-sitter-javascript/src/parser.c grammars/tree-sitter-javascript/src/scanner.c)
```
4. In the same CMakeLists.txt, don't forget to add "tspython" and "tsjavascript" as arguments at the bottom in target_link_libraries. 
5. In jsitter/native/grammars, git clone the tree-sitter-python and tree-sitter-javascript grammars (from https://github.com/tree-sitter/tree-sitter-python & https://github.com/tree-sitter/tree-sitter-javascript).
6. In jsitter, run ./make.sh (if you have errors on the .primary_state_ids field in parser.c, comment the line)
7. In jsitter, run mvn install
8. Copy the generated .so files to the polyglot project: cp jsitter/native/build/linux-x86-64/\*.so PolyglotAST/src/main/resources/
9. In the PolyglotAST folder, run mvn clean compile
10. Now you can run App.java and start developing on top of it.
11. If you wish to add support for more languages to the framework, repeat steps 3 through 9. A list of grammars is available under the "Parsers" section of this page: https://tree-sitter.github.io/tree-sitter/

# Classes of interest

## PolyglotTreeHandler

This class mainly acts as a wrapper for a tree-sitter object, that has methods to help handle polyglot-specific functions. 

The main functions to look at are `makePolyglotSubtree`, `isPolyglotEvalCall`, `isPolyglotImportCall` and `isPolyglotExportCall` as they are the ones most closely linked to polyglot features. 

`makePolyglotSubtree` handles the building and linking of a tree from a foreign code evaluation. It registers the code that would be executed, builds a subsequent polyglot tree handler to wrap it, and adds it to the list of linked nodes to allow traversing.

The other three functions mark nodes that are related to polyglot by returning a boolean. They are mainly a public interface for users of the framework, but require extension whenever a language is added.

## PolyglotZipper

This class acts as a wrapper for tree-sitter zipper object, which allows navigation of an AST through the `up`, `left`, `right` and `down` primitives. Those calls are reimplemented to allow similar usage of a polyglot AST without having to significantly acknowledge the polyglot nature of the AST, by leveraging the built polyglot tree handler it operates on; in addition, polyglot zippers provide functions to mark polyglot related nodes as such.

## Other classes

The other classes in the project give examples of services that could be implemented onto this framework in order to provide code analysis features. They are essentially dummy use-cases to demonstrate how to make use of the two main classes.

The repository also includes an App class with a runnable main function which gives another short example of how to use the framework.
