# CS 524 Final Project -- TCP in Maude

This repo is for the final project of UIUC 23SP CS 524. In this project, we are going to explore how to formalize TCP using Maude.

## How to run a test?

In your local repo, run the following command to start the Maude

```bash
./Maude3.3/maude.darwin64 # Start Maude 3.3
```

There are two major test methods, using `frew` or using `search`.
### Run `frew` test

In the Maude 3.3, run

```bash
load init_frew.maude
```

, which will setup the modules need to run a `frew` test.

Then you can run

```bash
load test_frew_l.maude
```

, which will run a test case with 101 messages from sender to receiver with a window size of 8.

If you want to see the whole trace of the `frew`, which is really large, we recommend you to start the Maude with the command line:

```bash
./Maude3.3/maude.darwin64 > test.out
```

, which will output all the result into a file test.out.
Once you start Maude in this way, you may run:

```bash
load test_frew_w_trace.maude
```

And this will show the detailed trace of the communication.

And to write your own test cases, you may refer to `test_frew_l.maude` and change the messages to send or window size. 

### Run `search` test

In the Maude 3.3, run

```bash
load init_search.maude
```

, which will setup the modules need to run a `frew` test.

To search for an terminating state of communication, you can run:

```bash
load search_to_end.maude
```

To search for an terminating state with massage recovery, you can run:

```bash
load search_to_end_w_resend.maude
```

To try searching for a bad state that the receiver get the wrong order of the massages, you can run:

```bash
load search_bad_state.maude
```

* Note: This may run for a relatively long time. On a Macbook Pro with M1 pro, it takes about 40 seconds.


## Code documentation 

### message-content

```Maude
fmod MESSAGE-CONTENT is
    sort MsgContent . --- message content, application-specific
endfm
```
Define the sort of `MsgContent`

### message-wrapper

```Maude
omod MESSAGE-WRAPPER is including MESSAGE-CONTENT .
    op msg_from_to_ : MsgContent Oid Oid -> Msg [ctor] .
endom
```

Define the construction of a message, which needs the content of massage, the Oid of the sender, and the Oid of the receiver.

### string_list

```Maude
fmod STRING-LIST is
    protecting LIST{String} * (sort NeList{String} to NeStringList ,
                             sort List{String} to StringList ) + INT .
    op mts : -> String [ctor] . 
    op EOF : -> String [ctor] . 

    op _[_] : StringList Int -> String .
    op pop(_,_) : StringList Int -> StringList .
    op assign(_,_,_) : StringList Int String -> StringList .
    op n_mts : Int -> StringList .

    var S : String . 
    var SL : StringList .
    var N : Int . 
    

    eq tail(nil) = nil .
    eq head(nil) = nil .

    eq SL [ N ] = if N <= 0 then if N == 0 then head(SL) else nil fi else tail(SL) [ N - 1 ] fi .
    eq pop(SL , N) = if N == 0 then tail(SL) else head(SL) (pop(tail(SL), N - 1)) fi .
    eq assign(SL, N, S) = if N < 0 or N >= size(SL) then SL else if N == 0 then S tail(SL) else head(SL) assign(tail(SL), N - 1, S) fi fi .
    eq n_mts(N) = if N <= 0 then nil else mts n_mts(N - 1) fi .
    
endfm
```

Define the sort of `StringList`.

* `mts`: stands for empty string, which can be used as a dummy placeholder in a `StringList`.
* `EOF`: stands for end of file, which is a special string needs to be placed at the end of message.
* `SL[N]`: return the `N`th string in the `StringList` `SL`.
* `pop(SL, N)`: pop the `N`th string in the `StringList` `SL`.
* `assign(SL, N, S)`: assign the `N`th string in the `StringList` `SL` to `S`.
* `n_mts(N)`: return a `StringList` of `N` empty strings.


### bool-list

```Maude
fmod BOOL-LIST is
    protecting LIST{Bool} * (sort NeList{Bool} to NeBoolList ,
                             sort List{Bool} to BoolList ) + INT .
                             
    op _[_] : BoolList Int -> Bool .
    op allTrue : BoolList -> Bool .
    op pop(_,_) : BoolList Int -> BoolList .
    op n_false : Int -> BoolList .
    op n_true : Int -> BoolList .
    op assign(_,_,_) : BoolList Int Bool -> BoolList .
    op assign(_,_:_,_) : BoolList Int Int Bool -> BoolList .
    op lastConsecutiveTrue : BoolList -> Int .

    var B : Bool .
    var BL : BoolList .
    var N N' : Int .    

    eq tail(nil) = nil .
    eq head(nil) = nil .
    eq allTrue(B) = B .
    eq allTrue(B BL) = allTrue(B) and allTrue(BL) .

    eq BL [ N ] = if N <= 0 then if N == 0 then head(BL) else nil fi else tail(BL) [ N - 1 ] fi .

    eq pop(BL , N) = if N == 0 then tail(BL) else head(BL) (pop(tail(BL), N - 1)) fi .

    eq n_false(N) = if N <= 0 then nil else false n_false(N - 1) fi .

    eq n_true(N) = if N <= 0 then nil else true n_true(N - 1) fi .

    eq assign(BL, N, B) = if N < 0 or N >= size(BL) then BL else if N == 0 then B tail(BL) else head(BL) assign(tail(BL), N - 1, B) fi fi .

    eq assign(BL, N : N', B) = if N < 0 or N' >= size(BL) or N > N' 
                                    then BL 
                                    else if N == 0 
                                         then B assign(tail(BL), 0 : N' - 1, B) 
                                         else head(BL) assign(tail(BL), N - 1 : N' - 1, B) 
                                         fi 
                                fi .
    eq lastConsecutiveTrue(BL) = if head(BL) == false or BL == nil then -1 else lastConsecutiveTrue(tail(BL)) + 1 fi .
endfm
```

Define the sort of `BoolList`.

* `_[_]`, `pop(_,_)` and `assign(_,_,_)` is the same as the ones in `StringList` except the fact that the element in the `List` is `Bool`.

* `allTrue(BL)`: return true if all the elements in the `BoolList` `BL` is `true`.
* `n_false(N)`: return a `BoolList` of `N` false.
* `n_true(N)`: return a `BoolList` of `N` true.
* `assign(BL,N : N', B)`: assign the `N`th to the `N'`th elements(both ends are included) to `B`. For example: `assign(false false false false,0 : 1,true) => true true false false`
* `lastConsecutiveTrue(BL)`: return the index of the last consecutive `true` from the beginning of the `BoolList` `BL`. If `BL` starts with `false`, return `-1`. For example, `lastConsecutiveTrue(true true false true true) => 1`, and `lastConsecutiveTrue(false true false true true) => -1`.

### TCP

#### OConfig

```Maude
sort OConfig .
subsort Object < OConfig < Configuration .
op __ : OConfig OConfig -> OConfig [ctor config assoc comm id: none] .
```
Define the sort of `OConfig`, which is a subsort of `Configuration`. `OConfig` represents a "soup" of `Object`s without any `Msg`.

#### Time_Configuration

```Maude
sort NatTime .
subsort Nat < NatTime .
sort Time_Configuration .

op { _ | _ } : Configuration NatTime -> Time_Configuration [ctor] .
```

Define the sort of `NatTime` and `Time_Configuration`. A `Time_Configuration` is a `Configuration` with a global time.

#### class Channel

* Note: This channel class is used in `frew` test. The one in `search` test is very similar except the modification that there is no randomness.

```Maude
class Channel | msgs : MsgList,
                count : Nat .

    crl [msg-loss] : < O : Channel | msgs :  ML , count : N >  
        => 
        < O : Channel | msgs : pop( ML , random(N) rem (size(ML)) ) , count : N + 1 > 
        if size(ML) > 0 and (random(N) rem 5 == 0) .
    
    crl [convey-msg] : < O : Channel | msgs :  ML , count : N >  
        => 
        < O : Channel | msgs : pop( ML , random(N) rem (size(ML)) ) , count : N + 1 > ((ML) [ random(N) rem (size(ML)) ]) 
        if size(ML) > 0 and not (random(N) rem 5 == 0) .
```


`Channel` class serves as the container of all messages during transmission, and will simulate the situation when a message is successfully transmitted or lost.

* `msgs`: All the messages need to be transmitted, including the messages from sender to receiver and the acknowledges from receiver to sender.

* `count`: A counter to calculate how many operations the `Channel` has run, which is served as the input of the random number generator.

#### class Sender

```
class Sender | msgsToSend : StringList, 
                sendBuffer : MsgList, 
                currentSeqNo : Nat, 
                receiver : Oid, 
                ackBuffer : BoolList, 
                numWordsInBuffer : Nat, 
                sendLast : Bool,
                finishSend : Bool, 
                channel : Oid, 
                bufferSize : Nat,
                bufferStart : Int ,
                lastSendMsgs : MsgList .


        rl [start] :
            < O : Sender | currentSeqNo : 0 , bufferSize : N >
            =>  
            < O : Sender | ackBuffer : n_true(N) , finishSend : false , bufferStart : 1 - N > .

        crl [load_word] : 
            < O : Sender | msgsToSend : S SL, sendBuffer : ML , numWordsInBuffer : N , currentSeqNo : N' , ackBuffer : AckL , finishSend : B , receiver : O', bufferSize : N0, bufferStart : I > 
            => 
            < O : Sender | msgsToSend : SL, sendBuffer : (msg (S withSeqNo s N') from O to O') ++ ML, numWordsInBuffer : s N , currentSeqNo : s N' , ackBuffer : n_false(N0) , bufferStart : I + 1 > if ((N == 0 and allTrue(AckL)) or (N < N0 and N > 0)) and not B .

        crl [load_dummy_word] : 
            < O : Sender | msgsToSend : nil, sendBuffer : ML , numWordsInBuffer : N , currentSeqNo : N' , ackBuffer : AckL, receiver : O', sendLast : false, bufferSize : N0 , bufferStart : I > 
            => 
            < O : Sender | msgsToSend : nil, sendBuffer : ML ++ (msg (mts withSeqNo s N') from O to O' ) , numWordsInBuffer : s N , currentSeqNo : s N' , ackBuffer : n_false(N0) , bufferStart : I + 1 > if (N == 0 and allTrue(AckL)) or (N < N0 - 1 and N > 0) .

        crl [load_last_dummy_word] : 
            < O : Sender | msgsToSend : nil, sendBuffer : ML , numWordsInBuffer : N , currentSeqNo : N' , sendLast : B0 ,finishSend : B , receiver : O', bufferSize : N0 , bufferStart : I > 
            => 
            < O : Sender | msgsToSend : nil, sendBuffer : ML ++ (msg (mts withSeqNo s N') from O to O' ) , numWordsInBuffer : s N , currentSeqNo : s N' , sendLast : true , ackBuffer : n_false(N0) , bufferStart : I + 1 > if N == (N0 - 1) .
        
        crl [sendCurrentMsgs] :
            < O : Sender | sendBuffer : ML , numWordsInBuffer : N', finishSend : B , bufferSize : N0 > < O' : Channel | msgs : ML0 , count : N > 
            =>  < O : Sender | sendBuffer : Mnil , numWordsInBuffer : 0 , lastSendMsgs : ML > < O' : Channel | msgs : (ML0 ++ ML) , count : 1 + N >  if ((N' == N0) and not B) .

        crl [rcvNewAck] : 
            (msg (ack withSeqNo N) from O' to O) < O : Sender | ackBuffer : AckL , bufferStart : N0 >
        => < O : Sender | ackBuffer : assign(AckL, 0 : (N - N0) , true) > if s N > N0 .

        crl [rcvTooOldAck] : 
            (msg (ack withSeqNo N) from O' to O) < O : Sender | currentSeqNo : N' >
        =>
            < O : Sender | > if s N < N' .


        crl [resendMsgs] : 
            { < O : Sender | ackBuffer : AckL , lastSendMsgs : ML , finishSend : false, sendBuffer : Mnil > < O' : Channel | msgs : Mnil , count : s N > R | NT }
        => 
            { < O : Sender |  > < O' : Channel | msgs : unackPacket(reverse(ML),AckL), count : N + 2 > R | NT + 1 } if not allTrue(AckL) .

        crl [finishSend] : 
            < O : Sender | sendLast : true, ackBuffer : AckL , finishSend : false, lastSendMsgs : ML >
        => 
            < O : Sender | finishSend : true , lastSendMsgs : Mnil > if allTrue(AckL) .

```

* `msgsToSend`: All the messages that has not been sent yet.
* `sendBuffer`: The buffer for the messages is going to sent in the window.
* `currentSeqNo`: The sequence number of the message that is going to be sent.
* `receiver`: The Oid of the receiver.
* `ackBuffer`: The buffer for the acknowledges received from the receiver.
* `numWordsInBuffer`: The number of messages in the `sendBuffer`.
* `sendLast`: A flag to indicate whether the last message has been sent.
* `finishSend`: A flag to indicate whether all the messages has been sent and get acknowledge.
* `channel`: The Oid of the channel.
* `bufferSize`: The size of the buffer.
* `bufferStart`: The sequence number of the first message in the buffer.
* `lastSendMsgs`: The messages that has been sent in the last send trial.
* `start`: Initialize the `Sender`.
* `load_word`: Load a word from `msgsToSend` to `sendBuffer`.
* `load_dummy_word`: Load a dummy word to `sendBuffer` if `msgsToSend` is empty.
* `load_last_dummy_word`: Load the last dummy word to `sendBuffer` and set the flag `sendLast` to be `true` to indicate that there is no need to further load any word to the `sendBuffer`.
* `sendCurrentMsgs`: Send the messages in the `sendBuffer` to the `Channel`.
* `rcvNewAck`: Receive a new acknowledge from the `Channel`.
* `rcvTooOldAck`: Receive an acknowledge that is too old to be useful.
* `resendMsgs`: Resend the messages in the `lastSendMsgs` to the `Channel` only when all the massages are consumed in the `Configuration` and the sender still do not receive all the acknowledges for the last window of messages it sent.
* `finishSend`: Set the flag `finishSend` to be `true` if all the messages are sent and all the acknowledges are received.


#### class Receiver

```Maude
class Receiver | greatestSeqNoRcvd : Nat,
                    sender : Oid,
                    msgsRcvd : StringList ,
                    rcvBuffer : StringList ,
                    rcvCheck : BoolList ,
                    channel : Oid ,
                    bufferSize : Nat ,
                    bufferStart : Int ,
                    rcvEOF : Bool ,
                    EOFPos : Nat .
        

        crl [loadWordsFromBuffer] :
            < O : Receiver | rcvBuffer : SL0, msgsRcvd : SL , rcvCheck : BL , greatestSeqNoRcvd : N , bufferSize : N0, bufferStart : I0 , rcvEOF : B0, EOFPos : N1 >
        => 
            < O : Receiver | rcvBuffer : n_mts(N0) , msgsRcvd :  SL SL0 , rcvCheck : n_false(N0), greatestSeqNoRcvd : N + N0, bufferStart : I0 + N0 > if (allTrue(BL) or (B0 and lastConsecutiveTrue(BL) >= N1)).

        crl [rmEmptyString] : 
            < O : Receiver | msgsRcvd : SL S0 >
        =>
            < O : Receiver | msgsRcvd : SL > if S0 == mts .

        crl [rcvOldPacket] :
            (msg (S withSeqNo N) from O' to O) < O : Receiver | greatestSeqNoRcvd : N' >
        =>
            < O : Receiver | > msg (ack withSeqNo N') from O to O' if N <= N' .

        crl [rcvNewPacket] :
            (msg (S withSeqNo N) from O' to O) < O : Receiver | greatestSeqNoRcvd : N' , bufferStart : N0 , bufferSize : N1 , rcvCheck : BL , rcvBuffer : SL > < O2 : Channel | count : N2, msgs : ML >
        =>
            < O : Receiver | rcvBuffer : assign(SL, N - N0, S), rcvCheck : assign( BL, N - N0, true) > < O2 : Channel | count : s N2, msgs : ML ++ msg (ack withSeqNo (N0 + lastConsecutiveTrue(assign( BL, N - N0, true )))) from O to O' >   if N >= N' /\ N < N0 + N1 /\ not (S == EOF) .

        crl [rcvEOFPacket] :
            (msg (EOF withSeqNo N) from O' to O) < O : Receiver | greatestSeqNoRcvd : N' , bufferStart : N0 , bufferSize : N1 , rcvCheck : BL , rcvBuffer : SL >
        =>
            < O : Receiver | rcvBuffer : assign(SL, N - N0, EOF), rcvCheck : assign( BL, N - N0, true ), rcvEOF : true, EOFPos : N - N0 > msg (ack withSeqNo (N0 + lastConsecutiveTrue(assign( BL, N - N0, true )))) from O to O' if N >= N' /\ N < N0 + N1 .

```

* `greatestSeqNoRcvd`: The greatest sequence number of the message that has been received.
* `sender`: The Oid of the sender.
* `msgsRcvd`: The messages that has been received with correct order.
* `rcvBuffer`: The buffer for the messages that has been received.
* `rcvCheck`: The buffer for checking which messages has been received.
* `channel`: The Oid of the channel.
* `bufferSize`: The size of the buffer.
* `bufferStart`: The sequence number of the first message in the buffer.
* `rcvEOF`: A flag to indicate whether the receiver has received the EOF message.
* `EOFPos`: The position of the EOF message in the `rcvBuffer` if it is received.
* `loadWordsFromBuffer`: Load the messages in the `rcvBuffer` to the `msgsRcvd` if all the messages in the `rcvBuffer` has been received.
* `rmEmptyString`: Remove the tailing empty strings in the `msgsRcvd`.
* `rcvOldPacket`: Receive a packet that is too old to be useful.
* `rcvNewPacket`: Receive a new packet, put it into the `rcvBuffer`, and send the acknowledge of the last packet this is received sequentially.
* `rcvEOFPacket`: Receive the EOF packet, put it into the `rcvBuffer`, and send the acknowledge of the last packet this is received sequentially.
