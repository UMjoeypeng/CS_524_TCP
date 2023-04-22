# CS 524 Final Project -- TCP in Maude

This repo is for the final project of UIUC 23SP CS 524. In this project, we are going to explore how to formalize TCP using Maude.

## How to run a unit test?

In your local repo, run the following command
```
./Maude3.3/maude.darwin64 # Start Maude 3.3
```

Then in Maude 3.3, run
```
load init.maude
```
which will load the necessary maude modules.

`test_fix_s.maude` and `test_fix_l.maude` are two test cases for the project.

You may run the following command to have a unit test.

```
set trace on .
load test_fix_s.maude # or load test_fix_l.maude
```
