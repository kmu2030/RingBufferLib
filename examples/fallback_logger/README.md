# About This Example
This example demonstrates log output.
It showcases logging by combining [RingBufferLib](https://github.com/kmu2030/RingBufferLib) with [FileStreamWriterLib](https://github.com/kmu2030/FileStreamWriterLib), [FileStreamReaderLib](https://github.com/kmu2030/FileStreamReaderLib), and [TcpClientLib](https://github.com/kmu2030/TcpClientLib).
This allows for simultaneous writing of logs to the controller's SD card and sending them to a remote TCP endpoint.

Furthermore, if sending to the TCP endpoint fails, it **falls back** to writing to the SD card.
When the TCP endpoint connection is restored,
the logs that were output during the fallback period are then sent to a dedicated TCP endpoint for fallback logs.

The logger uses its buffer as holding memory to retain values even after a power loss.
Additionally, when writing logs to the SD card or sending them to a remote TCP endpoint,
the output programs also refer to logs the logger wrote before those programs started.

## Operating Environment
To use this project, the following environment is required.

| Item           | Requirement       |
| :------------- | :---------------- |
| Controller     | NX or NJ          |
| Sysmac Studio  | Latest version recommended |

## Development Environment
This project was developed using the following environment.

| Item            | Version               |
| :-------------- | :-------------------- |
| Controller      | NX102-9000 Ver 1.64   |
| Sysmac Studio   | Ver.1.62              |
