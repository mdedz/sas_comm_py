# SAS Gaming Machine Communication

This package provides a robust system for communicating with gaming machines using the SAS Protocol. It supports tasks such as sending commands, receiving responses, and handling errors such as wrong CRC values. The package also includes a flexible task system with both one-shot and listener tasks for capturing events from the machines.

## Features

- **SAS Protocol Support:** Communicate with gaming machines via the SAS Protocol.
- **Task System:** Schedule both single-shot commands and continuous listener tasks.
- **CRC Validation:** Uses the CRC16K (Kermit) algorithm to ensure data integrity.
- **Serial Communication:** Simplifies serial port communication and response parsing.
- **Event Capture:** Provides a generator-based interface (`capture_events()`) to continuously poll and handle responses.

## Installation

Install the package using pip:

```bash
pip install sas_comm_py
```

## Usage

Below are some examples demonstrating how to use the package to communicate with a gaming machine.

### Basic Communication

First, create an instance of the `SlotMachine` class by specifying the COM port, baudrate, and other parameters:

```python
from sas_gaming_machine import SlotMachine

# Initialize the slot machine communication on a given COM port.
machine = SlotMachine(com_port="COM3", baudrate=19200, address=1, wakeup_bit=128)

# Send a basic command with type "S" (Standard) and optional data.
response = machine.write_type_S(command=0x01, optional_data=[0x02, 0x03])
print("Response:", response)
```

### Using Task System Listeners

The package supports two types of task scheduling:

- **Listeners:** Continuous tasks that poll for events.
- **Single-Shot Tasks:** One-time commands that are executed immediately.

#### Adding Listener Tasks

Add a listener task by specifying a command and any optional data. Listener tasks will be automatically polled within the `capture_events()` generator.

```python
# Add a listener task that will be polled continuously.
machine.add_listener(command=0x05, optional_data=["0A", "0B"], time=2)

# Start capturing events. This will yield responses from the machine continuously.
for response in machine.capture_events():
    print("Listener Response:", response)
    # Use a break condition or timeout as needed.
```

#### Adding Single-Shot Tasks

Single-shot tasks are executed only once and then removed from the task queue.

```python
# Add a single-shot task.
machine.add_one_task(command=0x07, optional_data=["0C"])

# Capture the next event to see the response of the single-shot task.
response = next(machine.capture_events())
print("Single-shot Response:", response)
```

## Under the Hood

### CRC Calculation and Validation

The package uses a modified CRC16K (Kermit) algorithm to calculate a 16-bit CRC. The `crc_calculate` function computes the CRC for outgoing commands, and the `validate_crc` method checks the incoming data for correctness. If the CRC does not match, a `WrongCRC` exception is raised.

### Serial Communication

The `SlotMachine` class uses the Python `serial` library for communication. When a command is written using methods like `write_type_R`, `write_type_S`, or `write_type_M`, the command is transformed into bytes, sent over the serial port, and then the response is read and processed accordingly.

## SAS Protocol for Gaming Machines

This library is specifically designed to work with gaming machines that use the SAS Protocol. The protocol defines how commands and responses are structured, including the use of a wakeup bit and the inclusion of CRC for data integrity. By using this package, you can:

- Send commands reliably to gaming machines.
- Automatically calculate and verify CRC to ensure correct data transmission.
- Schedule periodic polls and event captures using a flexible task system.

## Logging and Debugging

The package uses Python's built-in `logging` module. You can configure logging to help debug communication issues:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your improvements or bug fixes.

## License

This project is licensed under the MIT License.