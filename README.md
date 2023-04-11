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

You may run the following command to test a basic test, which Alice send a sentence to Bob.
```
set trace on .
rew [10] < "Alice" : Sender | msgsToSend : "Sequence" ++ "numbers" ++ "are" ++ "great" ++ "fun", currentMsg : nil, currentSeqNo : 0, receiver : "Bob" , ack : 0 > < "Bob" : Receiver | greatestSeqNoRcvd : 0, msgsRcvd : nil, sender : "Alice" , sendAck : false > .
```
