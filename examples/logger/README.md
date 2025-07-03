# About This Example
This example demonstrates log output combining [UdpClientLib](https://github.com/kmu2030/UdpClientLib) and [FileStreamWriterLib](https://github.com/kmu2030/FileStreamWriterLib), both of which utilize [RingBufferLib](https://github.com/kmu2030/RingBufferLib).

For log writing by the Logger, which uses RingBufferLib's ring buffer, UdpClientLib's UdpClient sends logs in real-time to a remote UDP endpoint, while FileStreamWriterLib's FileStreamWriter outputs them to a log file on the controller's SD card. The Logger's log writing operates within a primary task with a 2ms cycle. The approximate log size per entry is 68 bytes. While writing in each cycle is acceptable, even with a larger buffer capacity for FileStreamWriter, flushing to the SD card will occur at a considerable frequency.

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
