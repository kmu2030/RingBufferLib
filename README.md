# RingBufferLib
**RingBufferLib** is a ring buffer library designed for OMRON's NX/NJ series, targeting **BYTE** data types.

The **RingBuffer** in this library can be used in **multi-tasking environments without needing locks**,
provided there is only one primary entity for reading and one for writing to the buffer.
Additionally, it supports obtaining **differences by tracking ring buffer reads/writes** and by **comparing buffer information**.
There's some additional information in the Japanese article, "リングバッファの機能を彫り出す", available at https://zenn.dev/kitam/articles/546f4610e9f39f.

Below are basic read and write operations for the RingBuffer.

```iecst
// The RingBuffer needs to be initialized.
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);

// Returns FALSE if the ring buffer does not have enough writable space
// greater than the Size specified.
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 // Byte array that holds the bytes to write.
                 In:=iWriteData,
                 // Starting position of the bytes to write.
                 // Default is 0.
                 Head:=0,
                 // Number of bytes to write.
                 // Default is 0.
                 Size:=iWriteDataSize,
                 // Whether to overwrite unread bytes.
                 // Default is FALSE (Do not overwrite).
                 AllowOverwrite:=FALSE);


// Reads from the ring buffer, up to the maximum of Size bytes.
RingBuffer_read(Context:=iBufferContext,
                Buffer:=iBuffer,
                // Byte array into which the read bytes are written.
                Out:=iReadData,
                // Position at which to start writing bytes.
                // Default is 0.
                Head:=0,
                // Number of bytes to read. Default is 0.
                Size:=SizeOfAry(iReadData),
                // Number of bytes read.
                ReadSize=>iReadDataSize,
                // Whether an overflow occurred.
                Overflow=>iOverflow);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

iOk := iReadDataStr = 'data';
```

Beyond interactions with BYTE arrays, RingBuffer can also perform operations between **RingBuffers themselves**.
For example, you can transfer byte sequences between RingBuffers as shown below.

```iecst
RingBuffer_init(Context:=iSourceBufferContext,
                Buffer:=iSourceBuffer);
RingBuffer_init(Context:=iDestinationBufferContext,
                Buffer:=iDestinationBuffer);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iSourceBufferContext,
                 Buffer:=iSourceBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

RingBuffer_getReadableSize(Context:=iSourceBufferContext,
                           Size=>iSourceReadableSize);

// Transfers the specified 'Size' bytes from the source buffer to the destination buffer.
// Returns FALSE and does nothing if the writable bytes in the destination buffer are less than 'Size',
// or if the readable bytes in the source buffer are less than 'Size'.
RingBuffer_transfer(Source:=iSourceBufferContext,
                    // Byte array targeted by the source RingBufferContext.
                    SourceBuffer:=iSourceBuffer,
                    // Destination RingBufferContext.
                    Destination:=iDestinationBufferContext,
                    // Byte array targeted by the destination RingBufferContext.
                    DestinationBuffer:=iDestinationBuffer,
                    // Number of bytes to transfer.
                    Size:=iSourceReadableSize,
                    // Whether to overwrite unread bytes.
                    // Default is FALSE (Do not overwrite).
                    AllowOverwrite:=FALSE,
                    // Number of bytes transferred.
                    TransferSize=>iTransferSize);
                    
RingBuffer_read(Context:=iDestinationBufferContext,
                Buffer:=iDestinationBuffer,
                Out:=iReadData,
                Head:=0,
                Size:=SizeOfAry(iReadData),
                ReadSize=>iReadDataSize);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

iOk := iReadDataStr = 'data';
```

In addition to these, the library provides several other POUs essential for operations using ring buffers.
Examples can be found in `RingBufferLibExample.smc2`.

## Characteristics as a Ring Buffer
RingBuffer possesses the following characteristics as a ring buffer.

  * **Not usable indefinitely.**   
    This is because it internally uses write and read counts that continuously increment.   
    Assuming a write rate of 65535 bytes per 1ms, the usage limit would be reached in approximately 8900 years.

  * **Maximum buffer capacity is 65535 bytes.**   
    This is due to the maximum number of definable array elements being 65535.

  * **Buffer writes allow overflow (writing to unread areas).**  
    Unless specified by the user, overflow will not occur, but POUs that perform buffer writes can specify an option to allow overflow.

  * **Usable in multi-tasking without locks, only when there is one primary entity for reading and one for writing.**   
    This is because by using continuously incrementing write and read counts, there are no values that are commonly updated by both write and read processes.

## Operating Environment
To use this project, the following environment is required.

| Item            | Requirement             |
| :-------------- | :---------------------- |
| Controller      | NX or NJ                |
| Sysmac Studio   | Latest version recommended |

## Development Environment
This project was developed using the following environment.

| Item            | Version                |
| :-------------- | :--------------------- |
| Controller      | NX102-9000 Ver 1.64    |
| Sysmac Studio   | Ver.1.62               |

## Library Usage Procedure
To use the library (`RingBufferLib.slr`), follow these steps.

1.  **Reference `RingBufferLib.slr` in your project.**

2.  **Build the project and confirm there are no errors.**   
    The library uses namespaces.   
    Ensure there are no naming conflicts between identifiers in your project and the library's namespaces.

## About the Example Projects
The following Sysmac projects are provided as examples for RingBufferLib.

  * `RingBufferLibExample.smc2`   
    This provides examples of basic usage of RingBufferLib.   
    You can verify its operation by running it in the simulator.   
    It includes unit tests for RingBufferLib.

  * `examples\`   
    These are practical examples using RingBufferLib.   
    Each program is intended to run on a controller.

## About Obtaining RingBuffer Differences
RingBuffer supports obtaining differences by **tracking reads/writes** and by **comparing buffer information**.
Difference acquisition is performed using only the basic information required for the ring buffer's implementation,
without special mechanisms or additional data.
Therefore, while there are constraints based on buffer capacity and the timing of difference acquisition,
the intended operation can be expected in typical ring buffer usage.
Furthermore, these constraints can be treated as design tradeoffs, allowing you to select options based on your requirements.

Differences target **write differences** (writes to the buffer) and **read differences** (reads from the buffer).
Differences can be obtained when the value indicated by the disparity between two buffer information snapshots remains in the buffer.
Buffer information maintains write and read counts that continuously increment from its initialization through use.
Therefore, by comparing two buffer information snapshots, differences can always be obtained.
However, because the buffer is used circularly, whether the value indicated by the disparity remains in the buffer depends on the situation.

Differences are consumed by **overwriting the region holding the difference** through writes to the buffer.
Therefore, as long as the buffer is in use, the available period for obtaining read differences will always be shorter compared to write differences.
This is because write differences are consumed when writes equivalent to the buffer capacity occur,
whereas read differences are consumed when writes equivalent to the available capacity at the time of reading occur.

Additionally, difference acquisition is a **parallel process** to buffer operations.
Therefore, the buffer's state may change between the start and end of the process due to reads or writes occurring during that time.
POUs that perform difference acquisition return a value indicating whether there was a change in the buffer state; if a change occurred, they return FALSE.
Even if the buffer state changes during difference acquisition, in most cases, re-acquiring the difference will provide the current difference.
For continuous processing, carrying over to the next process might be acceptable.
However, it is crucial to note that **buffer writes can precede changes in buffer state**.
If a POU returns FALSE when acquiring differences from a very high-frequency circulating buffer,
the acquired difference might potentially include newly written values.
If such difference acquisition from the buffer is necessary,
you should either schedule it synchronously with operations on that buffer or increase the buffer capacity to handle it.

---
### Constraints on Difference Acquisition
In most of its applications, a ring buffer is not intended to overflow or circulate at very high frequencies.
Therefore, these constraints are unlikely to be actual limitations in most cases.
Moreover, all can be addressed by adjusting buffer capacity and through design ingenuity.

#### Constraints on Write Difference Acquisition
Write difference acquisition has the following constraints.

  * No writes exceeding the buffer capacity occurred during the comparison period.   
    Writes exceeding the buffer capacity mean writing into the area holding unacquired differences.

#### Constraints on Read Difference Acquisition
Read difference acquisition has the following constraints.

  * No writes exceeding the writable capacity at the beginning of the comparison period occurred.

  * No overflow occurred.

  * Even if an overflow occurred and was resolved during the comparison period, and normal reads were performed, those reads cannot be acquired.   
    Buffer operations and the resolution of overflows during the comparison period cannot be recognized.

#### Constraints on Re-acquiring Write Differences
Re-acquiring write differences has the following constraints.

  * No writes exceeding the buffer capacity occurred from the start of the difference acquisition process that returned FALSE.

#### Constraints on Re-acquiring Read Differences
Re-acquiring read differences has the following constraints.

  * No writes exceeding the number of bytes that were writable at the start of the difference acquisition process that returned FALSE occurred.

---
### Difference Acquisition POUs
The following POUs are used for difference acquisition.

  * `RingBuffer_createWriteTracker`   
    Creates a tracker to track buffer writes.

  * `RingBuffer_createReadTracker`   
    Creates a tracker to track buffer reads.

  * `RingBuffer_createTracker`   
    Creates a tracker to track both buffer writes and reads.

  * `RingBuffer_pullWrite`   
    Obtains write differences into a RingBuffer using the tracker created with `RingBuffer_createWriteTracker`.

  * `RingBuffer_pullRead`   
    Obtains read differences into a RingBuffer using the tracker created with `RingBuffer_createReadTracker`.

  * `RingBuffer_pull`   
    Obtains both write and read differences into a RingBuffer using the tracker created with `RingBuffer_createTracker`.

  * `RingBuffer_diffWrite`   
    Obtains write differences into a BYTE array by comparing buffer information.

  * `RingBuffer_diffRead`   
    Obtains read differences into a BYTE array by comparing buffer information.

  * `RingBuffer_diff`   
    Obtains both write and read differences into a BYTE array by comparing buffer information.

---
#### Write Tracking
Write tracking involves continuously obtaining differences about buffer writes.
You create a tracker with `RingBuffer_createWriteTracker`
and periodically obtain differences with `RingBuffer_pullWrite` to continuously track buffer writes.

Here's an example.

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
RingBuffer_init(Context:=iTrackWriteBufferContext,
                Buffer:=iTrackWriteBuffer);

// Create a tracker.
// The state at the time of its creation becomes the baseline for comparison.
RingBuffer_createWriteTracker(Target:=iBufferContext,
                              Tracker=>iWriteTracker);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// Get the difference from the tracker and put it into the ring buffer.
// If the difference is successfully retrieved,
// update the tracker and make the compared state the new baseline for comparison.
// Returns TRUE if the difference is successfully retrieved.
RingBuffer_pullWrite(Context:=iTrackWriteBufferContext,
                     // Byte array of the RingBuffer to which the obtained difference is written.
                     Buffer:=iTrackWriteBuffer,
                     // Tracking context.
                     Tracker:=iWriteTracker,
                     // RingBufferContext of the RingBuffer being tracked.
                     Tracked:=iBufferContext,
                     // Byte array of the RingBuffer being tracked.
                     TrackedBuffer:=iBuffer,
                     // Whether to overwrite unread bytes in the RingBuffer where the difference is written.
                     // Default is FALSE (Do not overwrite).
                     AllowOverwrite:=FALSE,
                     // Whether the retrieved difference has missings.
                     Missing=>iMissing,
                     // Number of bytes of difference obtained.
                     PullSize=>iPullSize);

RingBuffer_read(Context:=iTrackWriteBufferContext,
                Buffer:=iTrackWriteBuffer,
                Out:=iTrackData,
                Head:=0,
                Size:=SizeOfAry(iTrackData),
                ReadSize=>iTrackDataSize);
iTrackDataStr := AryToString(In:=iTrackData[0],
                             Size:=iTrackDataSize);

iOk := iTrackDataStr = 'data';
```

---
#### Read Tracking
Read tracking involves continuously obtaining differences about reads from the buffer.
You create a tracker with `RingBuffer_createReadTracker`
and periodically obtain differences with `RingBuffer_pullRead` to continuously track buffer reads.

Here's an example.

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
RingBuffer_init(Context:=iTrackReadBufferContext,
                Buffer:=iTrackReadBuffer);

// Create a tracker.
// The state at the time of its creation becomes the baseline for comparison.
RingBuffer_createReadTracker(Target:=iBufferContext,
                             Tracker=>iReadTracker);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);
RingBuffer_read(Context:=iBufferContext,
                Buffer:=iBuffer,
                Out:=iReadData,
                Head:=0,
                Size:=SizeOfAry(iReadData),
                ReadSize=>iReadDataSize);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

// Get the difference from the tracker and put it into the ring buffer.
// If the difference is successfully retrieved,
// update the tracker and make the compared state the new baseline for comparison.
// Returns TRUE if the difference is successfully retrieved.
RingBuffer_pullRead(Context:=iTrackReadBufferContext,
                    // Byte array of the RingBuffer to which the obtained difference is written.
                    Buffer:=iTrackReadBuffer,
                    // Tracking context.
                    Tracker:=iReadTracker,
                    // RingBufferContext of the RingBuffer being tracked.
                    Tracked:=iBufferContext,
                    // Byte array of the RingBuffer being tracked.
                    TrackedBuffer:=iBuffer,
                    // Whether to overwrite unread bytes in the RingBuffer where the difference is written.
                    // Default is FALSE (Do not overwrite).
                    AllowOverwrite:=FALSE,
                    // Whether the retrieved difference has missings.
                    Missing=>iMissing,
                    // Number of bytes of difference obtained.
                    PullSize=>iPullSize,
                    // Whether the RingBuffer tracked has overflowed.
                    Overflow=>iOverflow);

RingBuffer_read(Context:=iTrackReadBufferContext,
                Buffer:=iTrackReadBuffer,
                Out:=iTrackData,
                Head:=0,
                Size:=SizeOfAry(iTrackData),
                ReadSize=>iTrackDataSize);
iTrackDataStr := AryToString(In:=iReadData[0],
                             Size:=iReadDataSize);

iOk := iReadDataStr = 'data'
    AND iTrackDataStr = 'data';
```

---
#### Write Difference Acquisition
Write difference acquisition involves retaining the buffer information from a baseline point of comparison
and then acquiring the difference by comparing that baseline buffer information with buffer information from any arbitrary point in time. 
Use this when the output destination for the difference is not a RingBuffer,
or for RingBuffers where the likelihood of failed difference acquisition is high.
You acquire the difference with `RingBuffer_diffWrite`.
If you want to continuously acquire differences,
you can achieve gap-free difference acquisition by using the `ComparedContext` returned by the POU in the next call.

Here's an example.

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
// Save the state that will serve as the baseline for comparison.
iCompareContext := iBufferContext;

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// Get the write difference from the RingBufferContext being compared.
// If you want to continuously get differences,
// ensure RingBufferWrite returns TRUE and save the ComparedContext.
RingBuffer_diffWrite(Context:=iBufferContext,
                     Buffer:=iBuffer,
                     // RingBufferContext to compare and calculate the difference.
                     RefContext:=iCompareContext,
                     // Byte array that stores the difference.
                     Diff:=iDiff,
                     // Starting position of the byte array where the difference will be stored. Default is 0.
                     Head:=0,
                     // Number of bytes of the difference stored.
                     Size=>iDiffSize,
                     // Whether the retrieved difference has missing.
                     Missing=>iMissing,
                     // Current RingBufferContext being compared.
                     ComparedContext=>iComparedContext);
iDiffStr := AryToString(In:=iDiff[0],
                        Size:=iDiffSize);

iOk := iDiffStr = 'data';
```

---
#### Read Difference Acquisition
Read difference acquisition involves retaining the buffer information from a baseline point of comparison
and then acquiring the difference by comparing that baseline buffer information with buffer information from any arbitrary point in time. 
Use this when the output destination for the difference is not a RingBuffer,
or for RingBuffers where the likelihood of failed difference acquisition is high.
You acquire the difference with `RingBuffer_diffRead`.
If you want to continuously acquire differences,
you can achieve gap-free difference acquisition by using the `ComparedContext` output by the POU in the next call.

Here's an example.

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
// Save the state that will serve as the baseline for comparison.
iCompareContext := iBufferContext;

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// Get the read difference from the RingBufferContext being compared.
// If you want to continuously get differences,
// ensure RingBuffer_diffRead returns TRUE and save the ComparedContext.
RingBuffer_diffRead(Context:=iBufferContext,
                    Buffer:=iBuffer,
                    // RingBufferContext to compare and calculate the difference.
                    RefContext:=iCompareContext,
                    // Byte array that stores the difference.
                    Diff:=iDiff,
                    // Starting position of the byte array where the difference will be stored. Default is 0.
                    Head:=0,
                    // Number of bytes of the difference stored.
                    Size=>iDiffSize,
                    // Whether the retrieved difference has missing.
                    Missing=>iMissing,
                    // Whether the RingBuffer tracked has overflowed.
                    Overflow=>iOverflow,
                    // Current RingBufferContext being compared.
                    ComparedContext=>iComparedContext);
iDiffStr := AryToString(In:=iDiff[0],
                        Size:=iDiffSize);

iOk := iDiffStr = 'data';
```
