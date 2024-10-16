# Unified Streaming and Non-streaming Two-pass End-to-end Model for Speech Recognition

> Binbin Zhang
>
> 2020



## Abstract

* unify streaming and non-streaming in one model
* dynamic chunk-based attention strategy
* 可通过控制chunk_size进而控制inference的latency
* ctc后，使用rescore进行重打分

## U2

![image-20210708195954115](..\images\image-20210708195954115.png)

![image-20210708200318828](..\images\image-20210708200318828.png)

* 由于卷积需要看到left和right的信息，所以在chunk的时候，采用了casual convolution (**适合时序的因果卷积**)

