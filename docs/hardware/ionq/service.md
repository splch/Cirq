# IonQ API Service

IonQ's API provides a way to execute quantum circuits on IonQ's trapped ion quantum computers
or on cloud based simulators.  As of April 2021, this access is restricted to partners.
See [Access and Authentication](access.md) for details of access.

## Service class

The main entrance for accessing IonQ's API are instances of the `cirq_ionq.Service` class.
These objects need to be initialized with an api key, see
[Access and Authentication](access.md) for details.

The basic steps for running a quantum circuit in a blocking manner are:

1. Create a circuit to run.
2. Create a `cirq_ionq.Service` with proper authentication and endpoints.
3. Submit this circuit to run on the service and await the results of this call.
(Or alternatively use asynchronous jobs and processing)
4. Transform the results in a form that is most useful for your analysis.

Here is a simple example of this flow:

```python
import cirq
import cirq_ionq as ionq

# A circuit that applies a square root of NOT and then a measurement.
qubit = cirq.LineQubit(0)
circuit = cirq.Circuit(
    cirq.X(qubit)**0.5,            # Square root of NOT.
    cirq.measure(qubit, key='x')   # Measurement store in key 'x'
)

# Create a ionq.Service object.
# Replace API_KEY with your api key.
# Alternatively, if you have the IONQ_API_KEY environment
# variable set, you can omit specifying this api_key parameters.
service = ionq.Service(api_key=API_KEY)

# Run a program against the service. This method will block execution until
# the result is returned (determined by periodically polling the IonQ API).
result = service.run(circuit=circuit, repetitions=100, target='qpu')

# The return object of run is a cirq.Result object.
# From this object, you can get a histogram of results.
histogram = result.histogram(key='x')
print(f'Histogram: {histogram}')

# You can also get the data as a pandas frame.
print(f'Data:\n{result.data}')
```
This produces output (will vary due to quantum randomness!)

```
Histogram: Counter({0: 53, 1: 47})
Data:
    x
0   0
1   0
2   0
3   0
4   0
.. ..
95  1
96  1
97  1
98  1
99  1

[100 rows x 1 columns]
```

## Service options

In addition to the `api_key`, there are some other options which are
useful for configuring the service.  These are passed as arguments
when creating a `cirq_ionq.Service` object.

* `remote_host`: The location of the api in the form of an url. If this is None,
then this instance will use the environment variable `IONQ_REMOTE_HOST`. If that
variable is not set, then this uses `https://api.ionq.co/{api_version}`.
* `default_target`: this is a string of either `simulator` or `qpu`. By setting this you do not have to specify a target every time you run a job using `run`, `create_job` or via the `sampler` interface.  A helpful pattern is to create two services with defaults for the simulator and for the QPU separately.
* `api_version`: Version of the api. Defaults to 'v0.1'.
* `max_retry_seconds`: The API will pull with exponential backoff for completed jobs.  By specifying this you can change the number of seconds before this retry gives up.  It is common to set this to a very small number when, for example, wanting to fail fast, or to be set very long for long running jobs.

## Next steps

[Learn how to build circuits for the API](circuits.md)

[How to use the service API](jobs.md)

[Get information about QPUs from IonQ calibrations](calibrations.md)
