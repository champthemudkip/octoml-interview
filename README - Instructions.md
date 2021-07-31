# Netheria

In this challenge you will be tasked with laying out a frontend for the provided "netheria" server,
a hardware-aware (hard-aware, if you will) accelerator for deep learning models. From the provided
hardware list, clients can choose to "target" a particular platform with one of two operations:

- benchmark: benchmark the model on the target platform with no optimizations from the engine
- accelerate: optimize the model on the target platform using the specified engine

The design for this frontend can be found at:
https://www.figma.com/file/gfD44AXC632l4gIYpHiUWf/FE-Candidate-Takehome?node-id=0%3A1

For places where the design may be vague (e.g. with dropdown selectors) use your best
taste to implement default/reasonable behavior based on what you can glean from the
API spec described below.

The Netheria API server is hosted by us and can be reached at http://netheria.takehome.octoml.ai/ .

The Netheria API is described below.

## Disclaimer

This will be one of the first few times we issue this take-home to candidates. We've done some
vetting of the difficulty internally but probably have not foreseen all difficulties. If you find
that after a couple hours this assignment is too difficult, please do not hesitate to let Adelbert
know at achang@octoml.ai .

## Deliverable

Once you are done please package your source code into a tarball and send it back to us.

The primary intent of this challenge is for us to gain insight into you as an engineer and so we will be evaluating on
clean, documented, and tested code above all. **In short, this challenge will be used as a heuristic for the kind of code
you would write as a fellow Octonaut**. So if you use things like tests and comments in "real" code, you should probably
do so here as well. Feel free to use any framework, etc. you wish, be it Vue, React, Backbone, ...

One thing that will be especially important to document is how to run your frontend against a local
instantiation of the server, either by providing say, `npm` commands or by providing a Dockerfile and providing the
Docker commands to run.

Feel free to submit support requests, issues, etc. to Adelbert at achang@octoml.ai.

Also, please try not to spend more than 3-4 hours on this task - we understand that folks have lives outside of interviewing
with us and in general we want to be respectful of your time. Do not worry too much about being fully faithful to the provided design,
it is intended primarily as something for you to aim at. If anything, focus on the layout and the UX aspects of the design
as opposed to making sure every pixel and color is right - a rudimentary design is fine, not all buttons/toggles have to work, etc.
If you'd like, you may also provide a text document outlining your thought process, approach, and feedback about this problem.

## Netheria API

The following APIs are described by JSON Schema, the spec of which can be found here:
https://json-schema.org/

### /hardware
- Method: GET
- Arguments: N/A
- Returns: JSON list of hardware targets that can be selected by the frontend/clients

Each hardware target is described by the following JSON Schema:

```json
{
    "type": "object",
    "properties": {
        "provider": { "enum": ["AWS", "GCP", "Azure"] },
        "instance": { "type": "string" },
        "cpu":      { "type": "number" },
        "memory":   { "type": "number" }
    }
}
```

Example `curl` call:

```shell
$ curl http://netheria.takehome.octoml.ai/hardware | jq '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   515  100   515    0     0   502k      0 --:--:-- --:--:-- --:--:--  502k
[
  {
    "provider": "AWS",
    "instance": "m4.large",
    "cpu": 2,
    "memory": 8
  },
  {
    "provider": "AWS",
    "instance": "m4.xlarge",
    "cpu": 4,
    "memory": 16
  },
...
```

### /benchmark
- Method: POST
- Body: BenchmarkSpec serialized as JSON
- Returns: 200 if BenchmarkSpec is valid, 400 if not

BenchmarkSpec has the following JSON Schema:

```json
{
    "type": "object",
    "properties": {
        "engine":         { "enum": ["ONNX", "TVM"] },
        "num_trials":     { "type": "number" },
        "runs_per_trial": { "type": "number" }
    }
}
```

A valid BenchmarkSpec will have `num_trials * runs_per_trial <= 1000`.

Example `curl` calls:

```shell
# Good request
$ cat benchmark.json
{
  "engine": "ONNX",
  "num_trials": 10,
  "runs_per_trial": 10
}

$ curl -iX POST -H "Content-Type: application/json" --data @./benchmark.json http://netheria.takehome.octoml.ai/benchmark
HTTP/1.1 200 OK
content-type: application/json
content-length: 2
date: Thu, 22 Jul 2021 16:44:59 GMT

""



# Bad request, 1000 * 10 > 1000
$ cat benchmark.json
{
  "engine": "ONNX",
  "num_trials": 1000,
  "runs_per_trial": 10
}

$ curl -iX POST -H "Content-Type: application/json" --data @./benchmark.json http://netheria.takehome.octoml.ai/benchmark
HTTP/1.1 400 Bad Request
content-type: text/plain; charset=utf-8
content-length: 0
date: Thu, 22 Jul 2021 16:44:07 GMT
```

### /accelerate
- Method: POST
- Body: AccelerateSpec serialized as JSON
- Returns: 200 if AccelerateSpec is valid, 400 if not

AccelerateSpec has the following (approximate) JSON Schema:

```json
{
    "type": "object",
    "properties": {
        "engine": { "enum": ["ONNX", TVM (see below)]}
    }
}
```

The TVM engine spec has the following JSON Schema:

```json
{
    "type": "object",
    "properties": {
        "kernel_trials": { "type": "number" }
    }
}
```

A valid TVM spec will have `kernel_trials <= 2000`.

Example `curl` calls:

```shell
# Good ONNX request
$ cat accelerate.json
{
  "engine": "ONNX"
}

$ curl -iX POST -H "Content-Type: application/json" --data @./accelerate.json http://netheria.takehome.octoml.ai/accelerate
HTTP/1.1 200 OK
content-type: application/json
content-length: 2
date: Thu, 22 Jul 2021 16:48:06 GMT

""



# Good TVM request
$ cat accelerate.json
{
  "engine": { "TVM": { "kernel_trials": 2000 } }
}

$ curl -iX POST -H "Content-Type: application/json" --data @./accelerate.json http://netheria.takehome.octoml.ai/accelerate
HTTP/1.1 200 OK
content-type: application/json
content-length: 2
date: Thu, 22 Jul 2021 16:46:54 GMT

""



# Bad TVM request, kernel_trials > 2000
$ cat accelerate.json
{
  "engine": { "TVM": { "kernel_trials": 3000 } }
}

$ curl -iX POST -H "Content-Type: application/json" --data @./accelerate.json http://netheria.takehome.octoml.ai/accelerate
HTTP/1.1 400 Bad Request
content-type: text/plain; charset=utf-8
content-length: 0
date: Thu, 22 Jul 2021 16:46:25 GMT
```
