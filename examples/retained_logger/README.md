# About This Example
This example demonstrates log output combining [UdpClientLib](https://github.com/kmu2030/UdpClientLib) and [FileStreamWriterLib](https://github.com/kmu2030/FileStreamWriterLib), both of which utilize [RingBufferLib](https://github.com/kmu2030/RingBufferLib).

Although the processing content is the same as [examples/logger](https://github.com/kmu2030/RingBufferLib/tree/main/examples/logger), the Logger retains values across power loss by using its buffer as holding memory.
Additionally, when writing logs to the SD card or sending them to a remote UDP endpoint,
the output programs also refer to logs the logger wrote before those programs started.

## Operating Environment
To use this project, the following environment is required:

| Item           | Requirement       |
| :------------- | :---------------- |
| Controller     | NX or NJ          |
| Sysmac Studio  | Latest version recommended |

## Development Environment
This project was developed using the following environment:

| Item            | Version               |
| :-------------- | :-------------------- |
| Controller      | NX102-9000 Ver 1.64   |
| Sysmac Studio   | Ver.1.62              |
