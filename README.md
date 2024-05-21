<!-- omit from toc -->
# `bbt` README

 [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Alire](https://img.shields.io/endpoint?url=https://alire.ada.dev/badges/bbt.json)](https://alire.ada.dev/crates/bbt.html)

bbt is a simple tool to black box check the behavior of an executable (hence the name, bbt stands for *Black Box Tester*).  
The expected behavior is described using the [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) *Given* / *When* / *Then* usual pattern, in a simple Markdown format. 

It can be as simple as :
```md
## Scenario : Command line version option

- When I run `uut --version`
- Then the output contains `version 1.0`
```

bbt original feature is that this specification is directly executable : there is no use of a shell"ish" language, no glue code, no configuration file.

Just simple and readable English sentences.

### Tests are easy to run

And, to run the test :  
> bbt my_test.md

Or to run all the tests file in a tree :
> bbt -r tests

That's it.

bbt as no dependencies on external lib or tools (diff, for example), and aims at reducing uses of other tools, Makefile complexity, and platform dependency.  
*Describe behavior once, verify everywhere!*

### Tests are easy to write

I could have written **behavior is easy to describe**. There is no difference, there is no separate language or tool : the behavior description is executable.

Here is the magic : bbt understand some of the words in the step description (that is words afters *When*, *Then*, etc.). It has it's own simple [DSL](https://en.wikipedia.org/wiki/Domain-specific_language), with a vocabulary dedicated to test, with  keywords like *run*, *output*, *contains*, etc.

### Tests are easy to run
bbt as no dependencies on external lib or tools (diff tools, for example), and aims at reducing uses of other tools, Makefile complexity, and platform dependency.  
*Describe behavior once, verify everywhere!*
You don't have to learn it by heart : ask for a template file with  
`bbt -ct` (or `--create_template`)  
or ask for the grammar with  
`bbt -lg` (or `--list_grammar`)  


### Tests are Self documented
Your tests scenarios are already documented : you wrote them in Markdown, they will be nicely presented on github out of the box.

Your tests run results is generated by `bbt`, by just using the `-o` option, and is also a Markdown file. It's mainly some info related to the run (time, platform, etc.), and the list of tests with the result, each of them with a link to the scenario file : if a test fail, just click on the link and you are in the scenario.  

To see what it looks like, refer to [bbt own test](docs/) 

## Objective of the tool and limitations
bbt is targeting command line programs, and aims at being an easy and obvious way to run 90% of the boring usual tests on simples input / output, command line error, environment variable effects, etc.  
It is not meant for :
- UI testing or Web interaction 
- Complex file system stuff
- White box testing (obviously :-)), checking internal states, or extensive API testing.  
Note for example that implementing Gherkin *Scenario Outline*, with the table of inputs and corresponding expected outputs (*Examples*) isn't a priority from my point of view : this is probably useful for testing a mathematical function call, far less from a black box testing point of vue.  

It probably won't be the only test tool of your project, but that's highly dependant on the nature of your application.

## Objective of the project
Alternative tools exists, (Refer TBD).  
This project aim at exploring the ability to use plain english and get rid of scripting and language specific code.  
Let's see where that leads.

---------------------------------------------------------------------

## Basic Concepts

Basic concepts of bbt files are illustrated in the previous example :

```md
## Scenario : Command line version option

- When I run `uut --version`
- Then the output contains `version 1.0`
```

1. **the BDD usual keywords** : `Scenario`, *When*, *Then*, etc.  
bbt use a subset of the [Gherkin language](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language), in the [Markdown with Gherkin](https://github.com/cucumber/gherkin/blob/main/MARKDOWN_WITH_GHERKIN.md#markdown-with-gherkin) format.

2. [**bbt specifics  keywords**](#Keywords) : *run*, *output*, *contains*, etc.  
Here is an example with keywords in bold :  
**Given** there **is** **no** `.utt` **directory**  
**When** I **run** `uut --init`  
**Then** there **is** **no** **error**  
**And** **file** `.uut/config.ini` **contains** `lang=de`  
  
4. **glue word** : *I*, *the*  
As illustrated in the previous example, some words are ignored by bbt. Their only aim is to give users a way to read and write more natural english. This semi-formal language is an important bbt feature. As long as the language remains simple, the risk of ambiguity is low (Describing behavior is specifying, and you certainly don't want ambiguity when writing specifications).
   
5. [**code span** (in Markdown parlance)](https://spec.commonmark.org/0.31.2/#code-spans), that is text surrounded by backticks : `` `uut --version` ``, `` `version 1.0` ``  
bbt uses code span to express a command, a file or directory name or some expected output.

> [!WARNING]
> Till now, there is no ambiguity in the grammar between file and string : bbt is always able to understand what it's about.  
> To avoid ambiguity between file and directory, I add to take into account both `file` and `directory` keywords.  
> It is not excluded in the future that the Markdown syntax for files becomes mandatory instead of code span for the same reason : backtick would then be reserved to command or other strings, and File or dir would be written ``[my_file](my_file.md)``.  
> I'll try to avoid this evolution as it would be less natural to write, and this goes against project objectives.  
 
6. [**Fenced code block** (in Markdown parlance)](https://spec.commonmark.org/0.31.2/#fenced-code-blocks), that is lines between ``` or ~~~  
Fenced code block are used to specify multiline output or file content, as in: 

    ~~~md
    ## Scenario: Command line help

    - When I run `uut -h`
    - Then the output is
    ```
    uut [options] [-I directory]
    options :
    -h : help
    -r : recurse
    ```
    ~~~

---------------------------------------------------------------------

## Syntax

### [Gerkhin language subset](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language)
- *Feature*
- *Scenario* or *Example*
- *Given*
- *When*
- *Then*
- *And* or *But*

### bbt own DSL 

bbt keywords, including both the Gerkhin subset and bbt specifics keywords may be obtained with `bbt -lk` (`--list_keywords`).  
But more interesting, the grammar can be obtained throught the `-lg` (`--list_grammar`) option. 

Each Step is a one line sentence, with a basic "subject verb object" structure, starting with the preposition/adverb/conjunction (*Given*, *When*, *And*, etc.). 
Add some attribute adjectives (e.g. *empty*), and here we are.

Here is an excerpt from the grammar :
```
When             run              text|file --> RUN_CMD
```
- First, the keywords : here, `When` and `run`. 
- Then, some text or file name, between backtick.
- And, at the end, the resulting action.

Note that all other tokens will be ignored.  
You can write
```
When run `my_command -r`
```
or 
```
When I once more run the mighty `my_command -r`
```
that's the same. (Just don't exagerate on either way!)  

## Grammar 

**1. Given**  

  *Given* is used to check or create initial conditions.

  - ``Given the directory `dir1` ``  
  - ``Given the file `file_name` ``  
    ~~~
    ```
    line 1
    line 2
    line 3
    ```
    ~~~
    Return *success* if bbt could create the dir1 file or the file *file_name* containing the text in the code fenced lines.

    If there is already a *file_name*, the Step will fail. To get it overwritten with a new file or directory, add the `new` adverb :  
    ``Given the new directory `dir1` ``  
    `` Given the new file `file_name` ``
    ~~~
    ```
    line 1
    line 2
    line 3
    ```
    ~~~
    Note that unless using the --assume_yes option, user will be prompted to confirm deletions.
   
**2. When**  

  *When* has two main functions : checking that a file with some contents is available before run, and running  a command.

  - `` When I run `cmd` ``  
    Return *success* if *cmd* was run.  
    *failed* usually means that bbt couldn't find the cmd to run.

  - `` When I successfully run `cmd` ``  
    Return *success* if *cmd* was run **and** returned no error.  
  (This is a shortcut to avoid the usual following line `Then I get no error`).

**3. Then**  

  Then express the check to be done. It's either the run return code, the run output or a file content. Like for the Given steps, 

- `then I get error` or `no error`  
  Return *success* if the previously run command returned an error or no error code. 

- `` then output contains `text` ``  or `then error output contains ...`  
  Return *success* if *cmd* output contains `text`, on standard or on error output. 

- `` then output is `text` ``  or `then error output is ...`  
  Return *success* if *cmd* output is only `text``, on standard or on error output.   

- `` then I get `text` ``  
  This is a synonym of ``then output is `text` ``  

- `` then `file` contains `text` ``  
  Return *success* if *file* contains `text`, but not necessarily only that text. 
  
- `` then `file` is `text` ``  

  Return *success* if *file* contains **only** `text`. 
  Note that both *is* and *contains* may be followed by a multiline text :
  ~~~
  When `file` contains 
  ```
    line 1
    line 2
    line 3
  ```

### Rules list

Extracted with `bbt -lg` (--list_grammar)
```
Given                    is               file|dir  --> CHECK_FILE_EXISTENCE
Given                    is no            file|dir  --> CHECK_NO_FILE
Given          file|dir                             --> CREATE_IF_NONE
Given          file|dir                   text|file --> CREATE_IF_NONE
Given          file|dir  containing       text|file --> CREATE_NEW
Given existing text|file                            --> CHECK_FILE_EXISTENCE
When                     run              text|file --> RUN_CMD
When                     successfully run text|file --> RUN_WITHOUT_ERROR
Then                     get                        --> OUTPUT_IS
Then                     get              text|file --> OUTPUT_IS
Then                     get              error     --> ERROR_RETURN_CODE
Then                     get no           error     --> NO_ERROR_RETURN_CODE
Then           text|file contains                   --> FILE_CONTAINS
Then           text|file contains         text|file --> FILE_CONTAINS
Then           text|file is                         --> FILE_IS
Then           text|file is               text|file --> FILE_IS
Then           output    contains                   --> OUTPUT_CONTAINS
Then           output    contains         text|file --> OUTPUT_CONTAINS
Then           output    is                         --> OUTPUT_IS
Then           output    is               text|file --> OUTPUT_IS
```



---------------------------------------------------------------------

## More advanced feature and cool stuff

### Background
*bbt* supports a Background scenario, that is a special scenario that will be executed before the start of each following scenario.

```md
### Background :
  - Given there is no `config.ini` file

### Scenario : normal use cas
  - When I run `uut create config.ini` 
  - When I run `uut append "size=80x40" config.ini` 
  - Then `config.ini` should contains `"size=80x40"`

### Scenario : the last command does not meet expectation (test should fail)
  - When I run `uut -v` 
  - Then I should get no error
```

Note that in this case, the second scenario will fail because of the background (the first create a `config.ini` file), and would not fail without the background. 

Background may appears at the beginning, at document level, or at feature level, or both.
Before each scenario, the document background will be executed, and then the feature background.

### No more tests directory?
Think about it : tests description are in the docs/tests/ directory, and you want the result in the docs/ directory.  
You just have to run 
```
bbt -o docs/tests_results.md docs/tests/*.md
```
The title is kind of a provocation. Obviously, you need to run the tests somewhere and there will be residue. But as there is no input or output to keep in that directory (unless debugging the test himself), you can as well delete the execution dir after each run.


---------------------------------------------------------------------

## Behavior

### Blank lines and Case sensitivity

bbt ignore blank lines and casing when comparing actual with expected output.
This is the expected behavior in most case, but not always. 
Refer to the TDL.

### Execution

bbt is executed when you launch the command. All output files will be created here, and input file are expected here. 

bbt scenarii are Markdown files. So if you don't specify the precise file names, bbt will try to execute all md file.  
To see what files, use `bbt --list_files`, possibly with `--recurse`.  
(or the short form `bbt -lf -r`).

But if you specify the files, even using wildcards, like in `bbt tests/robustness*`, then bbt will consider that you know what you do, maybe you have a different naming convention, and will try to run each of them. So that you can name your file `.bbt`, or `.gmd` as you which.

As a special rule, two file will be ignored even if they are in the search path : the template file (bbt_template.md), and the output file if the -o option is used. The first is not supposed to be run, and the second is probably a consequence of a previous run. 

### Scenario files format

The BBT Markdown subset try to comply with [CommonMark Spec](https://spec.commonmark.org/), meaning that bbt always generate Common Mark compliant Markdown.
On the other hand, restrictions apply when writing bbt scenario.

#### Steps
Because the lexer is not able to make a difference between bullet lines in steps definition or bullet lines in normal text, there is limitations on where you can use it.
- *bbt* will consider bullet line starting with `-` as comment before the first "scenario" header. 
- *bbt* will consider all lines starting with `-` as Step within a scenario. As a consequence, **Don't use `-` in comments within a Scenario.**
- *bbt* will also consider line starting with the other official bullet markers (`*`and`+`) as comment, and **not steps line marker**, so that you can use those markers where you want without interfering with the lexer.  
Our simple suggestion : uses `-` for Steps and `*` for all bullet list in comments.

#### Headings
Only [ATX headings](https://spec.commonmark.org/0.31.2/#atx-headings) are supported, not [Setext headings](https://spec.commonmark.org/0.31.2/#setext-headings), meaning that you can write :
```
## Feature: My_Feature
```
but
```
Feature: My_Feature
-------------------
```
won't be recognized as a Feature.

---------------------------------------------------------------------

## Tips

### Understanding what he doesn't understand

Error messages provided by the lexer are not bullet proof (and it is likely that no special effort will be put on improving error messages in the future...).

For example, if you forget backticks on dir1 in :  
``- Given the directory dir1 ``  
It wont tells you "didn't you forget to "code fence" dir1?".  
It will just says :  
`Unrecognized step "Given the directory dir1"`

A good reflex in such a case is to ask *bbt* what did he understand from the test file, thanks to the -e (--explain) option.  
It will tell you something like :  
`GIVEN_STEP, UNKNOWN, Text = "Given the directory dir1"`  
meaning that the only thing he was able to conclude is that it's "Given" step.  
On the the adjusted version :  
``GIVEN_STEP, FILE_CREATION, Text = "Given the directory `dir1`", File_Name = "dir1"``  
you see that the (internal) action field has changed from UNKNOWN to FILE_CREATION, and that the File_Name field has now a value.

  

---------------------------------------------------------------------

## TDL

**near future**
- given file doest not exists
- Given execution directory `dir1`  
- interactive exec
  
**distant future or low priority**
- environment
- diff 
- append 
- implement "case insensitive" and "ignore blank lines" 
- clean function : bbt delete all files created during execution
- explore the possibility to run multiple exe asynchronously, while staying simple.
  Maybe by using the AdaCore spawn lib.
- "should be" as "is" synonym?
    
---------------------------------------------------------------------

## Help and comments

Comments are welcome [here](https://github.com/LionelDraghi/bbt/discussions)

