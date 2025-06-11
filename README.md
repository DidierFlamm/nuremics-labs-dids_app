<img src="https://raw.githubusercontent.com/julien-siguenza/nuremics-data/main/assets/banner.jpg" alt="NUREMICS Banner" width="100%">

# NUREMICS® _LABS_

**NUREMICS® is an open-source Python™ framework for developing software-grade scientific workflows.**

🧠 Code like a scientist — build like an engineer.<br>
🧩 Modular workflows — no more tangled scripts.<br>
🧪 Parametric exploration — configuration over code.<br>
💾 Full traceability — everything written to disk.<br>
🛠️ Industrial mindset — R&D speed, software rigor.

## Foreword

The **NUREMICS®** project is organized into two complementary repositories:

- **`nuremics`**:
This repository is the core Python library, installable via `pip install`. It provides the foundational components to create modular and extensible software workflows.

- **`nuremics-labs`** _(current repository)_:
This repository contains examples of end-user applications built using the **NUREMICS®** framework. It is intended to be **forked** by developers to initiate their own `nuremics-labs` project and build custom applications tailored to their specific use cases.

Readers are invited to begin their exploration of the **NUREMICS®** project with the [`nuremics`](https://github.com/nuremics/nuremics) repository, to understand the core framework and its foundational concepts. Once you're familiar with the underlying logic, this `nuremics-labs` repository will guide you deeper into the code, showing how to develop your own **NUREMICS®** applications, and operate them both as a developer and as an end-user.

## Installation

Once the present `nuremics-labs` repository has been forked and cloned to your local machine, installation proceeds as follows:

1. **(Recommended) Create a dedicated coding environment.** While not mandatory, using [micromamba](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html) is highly recommended for fast, reproducible, and lightweight environment management:

```bash
micromamba create -n nrs-env python=3.12
micromamba activate nrs-env
```

2. **Install the required dependencies.** Use `pip` to install the packages listed in the `requirements.txt` file. If you need additional packages to support domain-specific application development, remember to add them to this file before running the command:

```bash
pip install -r requirements.txt
```

3. **Set the environment variable.** Add the absolute path of the `src` directory from your cloned `nuremics-labs` repository to the `PYTHONPATH` environment variable of your local system. If `PYTHONPATH` does not already exist, please create it. This allows Python to locate all project modules correctly.

```bash
/absolute/path/to/nuremics-labs/src
```

4. **(Optional) Define a default working directory.** It is also suggested to define another environment variable named `WORKING_DIR`, which serves as the default root folder where all your **Apps** write their results. This becomes particularly useful when working on multiple **Apps**, as it allows you to consistently manage output locations across your projects.

```bash
/absolute/path/to/default/working_dir
```

You're now ready to begin your coding journey with **NUREMICS®** 🧬

## Create App

This section walks you through the process of building a custom **NUREMICS®** application from scratch. You'll start by implementing your own **Procs**, which encapsulate domain-specific logic and computational tasks. Then, you’ll learn how to assemble these building blocks into a fully operational **App**, ready to run studies and generate structured results.

Whether you're developing a quick prototype or a full-scale scientific workflow, this guide will help you translate your ideas into modular, reusable, traceable and scalable software components.

### Implement Procs

We start by defining the core building blocks of the **App** to be created: the **Procs**. Each **Proc** is a reusable item that encapsulates a specific piece of logic executed within the overall workflow. Internally, this logic can be further decomposed into elementary operations, implemented as individual functions (units) within the **Proc** itself.

To implement our first **Proc**, we begin by importing the `Process` base class from **NUREMICS®**, which all custom **Procs** must inherit from. To make this inheritance simple and structured, we also import the `attrs` library, which helps define clean, data-driven Python classes.

```python
import attrs
from nuremics import Process
```

We then declare our first **Proc** as a Python class named `OneProc`, inheriting from the `Process` base class. This marks it as a modular item of computation which can be executed within a **NUREMICS®** workflow.

```python
import attrs
from nuremics import Process

@attrs.define
class OneProc(Process):
```

We now declare the input data required by our `OneProc`, grouped into two categories: **Parameters** and **Paths**. Each input is defined using `attrs.field()` and marked with `metadata={"input": True}`.

This metadata is essential: it tells the **NUREMICS®** framework that these attributes are expected as input data, ensuring they are properly tracked and managed throughout the workflow.

```python
import attrs
from pathlib import Path
from nuremics import Process

@attrs.define
class OneProc(Process):

    # Parameters
    param1: float = attrs.field(init=False, metadata={"input": True})
    param2: int = attrs.field(init=False, metadata={"input": True})
    
    # Paths
    path1: Path = attrs.field(init=False, metadata={"input": True}, converter=Path)
```

In addition to the previously declared input data, a **Proc** can also define internal variables: attributes used during the execution of its internal logic but not provided as input data.

These internal variables, like `variable` in our example below, are declared without the `metadata={"input": True}` tag, signaling to the **NUREMICS®** framework that they are not exposed to the workflow and will be set or computed within the **Proc** itself.

```python
import attrs
import pandas as pd
from pathlib import Path
from nuremics import Process

@attrs.define
class OneProc(Process):

    # Parameters
    param1: float = attrs.field(init=False, metadata={"input": True})
    param2: int = attrs.field(init=False, metadata={"input": True})
    
    # Paths
    path1: Path = attrs.field(init=False, metadata={"input": True}, converter=Path)

    # Internal
    variable: pd.DataFrame = attrs.field(init=False)
```

The operations executed by the **Proc** are finally implemented as elementary functions, which are then sequentially called within the `__call__()` method to define the overall logic of the **Proc**.

```python
import attrs
import pandas as pd
from pathlib import Path
from nuremics import Process

@attrs.define
class OneProc(Process):

    # Parameters
    param1: float = attrs.field(init=False, metadata={"input": True})
    param2: int = attrs.field(init=False, metadata={"input": True})
    
    # Paths
    path1: Path = attrs.field(init=False, metadata={"input": True}, converter=Path)

    # Internal
    variable: pd.DataFrame = attrs.field(init=False)

    def __call__(self):
        super().__call__()

        self.operation1()
        self.operation2()
    
    def operation1(self):
        # </> your code </>

    def operation2(self):
        # </> your code </>
```

Note that the **Proc** should at some point produce output data, typically in the form of files or folders generated during the execution of its operations. To make these output data trackable by the **NUREMICS®** framework, each must be registered in the `self.output_paths` dictionary using a label that is unique to the **Proc** (e.g., `"out1"`, `"out2"`).

Using the dictionary syntax `self.output_paths["out1"]` effectively declares an output variable named `out1`, which will later be instantiated by assigning it a specific file or folder name when integrating the **Proc** into a broader application workflow.

```python
import attrs
import pandas as pd
from pathlib import Path
from nuremics import Process

@attrs.define
class OneProc(Process):

    # Parameters
    param1: float = attrs.field(init=False, metadata={"input": True})
    param2: int = attrs.field(init=False, metadata={"input": True})
    
    # Paths
    path1: Path = attrs.field(init=False, metadata={"input": True}, converter=Path)

    # Internal
    variable: pd.DataFrame = attrs.field(init=False)

    def __call__(self):
        super().__call__()

        self.operation1()
        self.operation2()
    
    def operation1(self):
        # </> your code </>
        file = self.output_paths["out1"]
        # </> Write file </>

    def operation2(self):
        # </> your code </>
        file = self.output_paths["out2"]
        # </> Write file </>
```

Even though **Procs** are not intended to be executed independently by end-users, they are still designed with the possibility to run _out of the box_. This allows developers to easily execute them during the development phase or when implementing dedicated unit tests for a specific **Proc**.

In such cases, it is important to set `set_inputs=True` when instantiating the **Proc**, to explicitly inform the **NUREMICS®** framework that the input data are being provided manually, outside of any workflow context.

```python
import attrs
import pandas as pd
from pathlib import Path
from nuremics import Process

@attrs.define
class OneProc(Process):

    # Parameters
    param1: float = attrs.field(init=False, metadata={"input": True})
    param2: int = attrs.field(init=False, metadata={"input": True})
    
    # Paths
    path1: Path = attrs.field(init=False, metadata={"input": True}, converter=Path)

    # Internal
    variable: pd.DataFrame = attrs.field(init=False)

    def __call__(self):
        super().__call__()

        self.operation1()
        self.operation2()
    
    def operation1(self):
        # </> your code </>
        file = self.output_paths["out1"]
        # </> Write file </>

    def operation2(self):
        # </> your code </>
        file = self.output_paths["out2"]
        # </> Write file </>

if __name__ == "__main__":
    
    # Define working directory
    working_dir = ...

    # Go to working directory
    os.chdir(working_dir)

    # Create dictionary containing input data
    dict_inputs = {
        "param1": ...,
        "param2": ...,
        "path1": ...,
    }
    
    # Create process
    process = OneProc(
        dict_inputs=dict_inputs,
        set_inputs=True,
    )
    process.output_paths["out1"] = "output1.csv"
    process.output_paths["out2"] = "output2.png"

    # Run process
    process()
    process.finalize()
```

**Note:** The complete implementation of the `OneProc` **Proc**, as well as that of the `AnotherProc` **Proc** later used in this tutorial, can be found in the repository under [`src/procs/OneProc`](https://github.com/nuremics/nuremics-labs/tree/main/src/procs/OneProc) and [`src/procs/AnotherProc`](https://github.com/nuremics/nuremics-labs/tree/main/src/procs/AnotherProc), respectively.

### Assemble Procs into App

Most of the development effort has already been carried out when implementing the individual **Procs**. The next step consists in assembling them into a coherent **App**, where each **Proc** is instantiated, connected, and orchestrated to form a complete, executable workflow.

We start by defining the name of our **App**.

```python
APP_NAME = "ONE_APP"
```

We then import the `Application` class from the **NUREMICS®** framework, which serves as the container and manager to define a workflow composed of multiple **Procs**.

```python
APP_NAME = "ONE_APP"

from nuremics import Application
```

We now import the two **Procs**, `OneProc` and `AnotherProc`, that we previously implemented. These will be the building blocks to assemble into our final **App**.

```python
APP_NAME = "ONE_APP"

from nuremics import Application

from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc
```

The source code of the **App** then adopts the structure of a standard Python script, which can both be executed directly or imported as a module. This is achieved by defining a `main()` function and guarding it with the typical `if __name__ == "__main__":` statement.

```python
APP_NAME = "ONE_APP"

from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main():
    # Application logic here

if __name__ == "__main__":
    main()
```

In the `main()` function, we add two input arguments that the end-user must specify when launching the **App** inside the `if __name__ == "__main__":` block:

- `working_dir`: the working directory from which the **App** will be executed.

- `studies`: a list of study names that the end-user wants to perform with the **App**.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # Application logic here

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

Inside the `main()` function, we define a list called `workflow` which contains the sequence of **Procs** to be executed, in the order specified. This list is made up of dictionaries, where each dictionary describes the characteristics of a particular **Proc**.

Let's first define the key `"process"` of each dictionary, which specifies the **Proc** class (previously imported, e.g., `OneProc` and `AnotherProc`) to instantiate and execute within the **App** workflow.

This dictionary-based structure offers flexibility to easily add more parameters or options later by simply adding new keys to each dictionary in the workflow.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # --------------- #
    # Define workflow #
    # --------------- #
    workflow = [
        {
            "process": OneProc,
        },
        {
            "process": AnotherProc,
        },
    ]

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

We now create an `Application` object `app`, which acts as the core engine of our **App**. This object is instantiated using the previously defined inputs:

- `app_name`: the name of the **App**.

- `working_dir`: the root directory from which the **App** is executed.

- `workflow`: the ordered list of **Procs** to run.

- `studies`: the list of studies the end-user wishes to perform.

Once the `Application` object is created, calling `app()` launches the workflow execution of all the defined **Procs** for each study.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # --------------- #
    # Define workflow #
    # --------------- #
    workflow = [
        {
            "process": OneProc,
        },
        {
            "process": AnotherProc,
        },
    ]

    # ------------------ #
    # Define application #
    # ------------------ #
    app = Application(
        app_name=APP_NAME,
        working_dir=working_dir,
        workflow=workflow,
        studies=studies,
    )
    # Run it!
    app()

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

When running the **App**, **NUREMICS®** first provides the following terminal feedback:

- A visual banner indicating the launch of a **NUREMICS® App**.

- A structured overview of the assembled workflow, showing each registered **Proc**, its associated operations (functions), and their order of execution within the **App** workflow.

👤🔄🖥️
```shell
                                                              0000000000000000000000
                                      000                  00000000000000000000000000
                                       00000            0000000000000000000000000000000
                                        000000        0000000000000000000000000000000000
                                        0000000      00000000000000000000000000000000000
                                         0000000    000000000000000000000000000000000000
                                         0000000    0000000000000000000000000000000000000
                                         000000  00  00000000000000000000000000000000000
                                        000000  0000  0000000000000000000000000000000000
                                        000000  0000  0000000000000000000000000000000000
                                       000000  000000  00000000000000000000000000000000
                                      0000000  000000  000000000000000000000000000000
                                    00000000  00000000 0000000000000000000000000000
                                  0000000000  00000000  000000000000000000000000
                               0000000000000  00000000  00000000000000
                    000000000000000000000000  00000000  0000000000
                 000000000000000000000000000  00000000  00000000
               000000000000000000000000000000  000000  0000000
              0000000000000000000000000000000  000000  000000
             000000000000000000000000000000000  0000  000000
            0000000000000000000000000000000000  0000  000000
            00000000000000000000000000000000000  00  000000
           0000000000000000000000000000000000000    0000000
           00000000000000000000000000000000000000  00000000
            00000000000000000000000000000000000      0000000
            0000000000000000000000000000000000        000000
             0000000000000000000000000000000            00000
              000000000000000000000000000                  000
                00000000000000000000000                      0
                   000000000000000

> APPLICATION <

| Workflow |
ONE_APP_____
            |_____OneProc_____
            |                 |_____operation1
            |                 |_____operation2
            |
            |_____AnotherProc_____
                                  |_____operation1
                                  |_____operation2
                                  |_____operation3
```

At this stage, **NUREMICS®** also performs a structural check of each **Proc** by inspecting its `__call__` method. Specifically, it ensures that only functions defined within the **Proc** class itself are invoked during execution. This design choice enforces a clean and self-contained structure for each **Proc**, where all internal logic remains encapsulated.

Let’s consider a case where the developer does not adhere to this enforced structural rule, for instance, by injecting additional logic directly into the `__call__` method of a **Proc** (in this example, in the `AnotherProc` class).

```python
    def __call__(self):
        super().__call__()

        some_parameter = 2 # <-- External logic added here

        self.operation1()
        self.operation2()
        self.operation3()
```

In this situation, **NUREMICS®** will immediately raise a structural validation error and halt execution.

👤🔄🖥️
```shell
| Workflow |
ONE_APP_____
            |_____OneProc_____
            |                 |_____operation1
            |                 |_____operation2
            |
            |_____AnotherProc_____(X)

(X) Each process must only call its internal function(s):

    def __call__(self):
        super().__call__()

        self.operation1()
        self.operation2()
        self.operation3()
        ...
```

**NUREMICS®** is then expected to display a summary of all required input/output data for each **Proc**, along with their current mapping status within the **App**.

At this stage, the system automatically verifies whether every required input/output data has been properly mapped within the **App** configuration.

If any **input parameters** are missing, they are explicitly listed, and the developer is prompted to define them using either the `"user_params"` or `"hard_params"` key.

👤🔄🖥️
```shell
| OneProc |
> Input Parameter(s) :
(float) param1 -----||----- Not defined (X)
(int)   param2 -----||----- Not defined (X)

(X) Please define all input parameters either in "user_params" or "hard_params".
```

The **input parameters** of the **Proc** `OneProc` can be properly mapped within the **App** by defining the `"user_params"` and/or `"hard_params"` keys in its corresponding dictionary entry inside the `workflow` list.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # --------------- #
    # Define workflow #
    # --------------- #
    workflow = [
        {
            "process": OneProc,
            "user_params": {
                "param2": "parameter1",
            },
            "hard_params": {
                "param1": 0.5,
            },
        },
        {
            "process": AnotherProc,
        },
    ]

    # ------------------ #
    # Define application #
    # ------------------ #
    app = Application(
        app_name=APP_NAME,
        working_dir=working_dir,
        workflow=workflow,
        studies=studies,
    )
    # Run it!
    app()

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

When running the **App** again, **NUREMICS®** detects that all required **input parameters** for `OneProc` have been successfully mapped.

However, it now reports that one or more **input paths** are missing. These are explicitly listed, and the developer is prompted to define them using either the `"user_paths"` or `"required_paths"` key.

👤🔄🖥️
```shell
| OneProc |
> Input Parameter(s) :
(float) param1 -----||----- 0.5        (hard_params)
(int)   param2 -----||----- parameter1 (user_params)
> Input Path(s) :
path1 -----||----- Not defined (X)

(X) Please define all input paths either in "user_paths" or "required_paths".
```

The **input paths** of the **Proc** `OneProc` can be properly mapped within the **App** by defining the `"user_paths"` and/or `"required_paths"` keys in its corresponding dictionary entry inside the workflow list.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # --------------- #
    # Define workflow #
    # --------------- #
    workflow = [
        {
            "process": OneProc,
            "user_params": {
                "param2": "parameter1",
            },
            "hard_params": {
                "param1": 0.5,
            },
            "user_paths": {
                "path1": "input1.txt",
            },
        },
        {
            "process": AnotherProc,
        },
    ]

    # ------------------ #
    # Define application #
    # ------------------ #
    app = Application(
        app_name=APP_NAME,
        working_dir=working_dir,
        workflow=workflow,
        studies=studies,
    )
    # Run it!
    app()

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

When running the **App** again, **NUREMICS®** detects that all required **input paths** for `OneProc` have been successfully mapped.

However, it now reports that one or more **output paths** are missing. These are explicitly listed, and the developer is prompted to define them using the `"output_paths"` key.

👤🔄🖥️
```shell
| OneProc |
> Input Parameter(s) :
(float) param1 -----||----- 0.5        (hard_params)
(int)   param2 -----||----- parameter1 (user_params)
> Input Path(s) :
path1 -----||----- input1.txt (user_paths)
> Output Path(s) :
out1 -----||----- Not defined (X)
out2 -----||----- Not defined (X)

(X) Please define all output paths in "output_paths".
```

The **output paths** of the **Proc** `OneProc` can be properly mapped within the **App** by defining the `"output_paths"` key in its corresponding dictionary entry inside the workflow list.

In the same way, we also complete the mapping for the **Proc** `AnotherProc` by providing all required entries: `"user_params"` and/or `"hard_params"`, `"user_paths"` and/or `"required_paths"`, `"output_paths"`.

```python
APP_NAME = "ONE_APP"

from pathlib import Path
from nuremics import Application
from procs.OneProc.item import OneProc
from procs.AnotherProc.item import AnotherProc

def main(
    working_dir: Path = None,
    studies: list = ["Default"],
):
    # --------------- #
    # Define workflow #
    # --------------- #
    workflow = [
        {
            "process": OneProc,
            "user_params": {
                "param2": "parameter1",
            },
            "hard_params": {
                "param1": 0.5,
            },
            "user_paths": {
                "path1": "input1.txt",
            },
            "output_paths": {
                "out1": "output1.csv",
                "out2": "output2.png",
            },
        },
        {
            "process": AnotherProc,
            "user_params": {
                "param1": "parameter2",
                "param2": "parameter3",
            },
            "user_paths": {
                "path1": "input2.json",
                "path2": "input3",
            },
            "required_paths": {
                "path3": "output1.csv",
            },
            "output_paths": {
                "out1": "output3",
            },
        },
    ]

    # ------------------ #
    # Define application #
    # ------------------ #
    app = Application(
        app_name=APP_NAME,
        working_dir=working_dir,
        workflow=workflow,
        studies=studies,
    )
    # Run it!
    app()

if __name__ == "__main__":

    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

With all required mappings now properly defined for each **Proc**, the **App** can be executed without raising any errors. **NUREMICS®** confirms that the full mapping is complete by prompting a summary for each **Proc**, indicating that all **input parameters**, **input paths**, and **output paths** have been successfully resolved.

👤🔄🖥️
```shell
| OneProc |
> Input Parameter(s) :
(float) param1 -----||----- 0.5        (hard_params)
(int)   param2 -----||----- parameter1 (user_params)
> Input Path(s) :
path1 -----||----- input1.txt (user_paths)
> Output Path(s) :
out1 -----||----- output1.csv (output_paths)
out2 -----||----- output2.png (output_paths)

| AnotherProc |
> Input Parameter(s) :
(float) param1 -----||----- parameter2 (user_params)
(float) param2 -----||----- parameter3 (user_params)
> Input Path(s) :
path1 -----||----- input2.json (user_paths)
path2 -----||----- input3      (user_paths)
path3 -----||----- output1.csv (required_paths)
> Output Path(s) :
out1 -----||----- output3 (output_paths)
```

As the **App** has now been fully assembled, **NUREMICS®** displays a clean summary of its I/O interface, as it will appear to the end-user.

This summary includes all declared user parameters and user paths required as inputs, along with the corresponding output files and folders that the **App** will generate. It serves as an explicit interface contract, allowing end-users to clearly understand what data they need to provide and what results to expect.

👤🔄🖥️
```shell
> INPUTS <

| User Parameters |
> parameter1 (int)
> parameter2 (float)
> parameter3 (float)

| User Paths |
> input1.txt
> input2.json
> input3

> OUTPUTS <

> output1.csv
> output2.png
> output3
```

With all **Procs** implemented and properly assembled within the **App**, the development work is now complete. The developer’s responsibility ends here (excluding, of course, the implementation of unit tests to ensure long-term maintainability, which falls outside the scope of this tutorial).

The **App** is now fully functional and ready to be operated by end-users. From this point, users can interact with the **App** through its declared I/O interface, without needing to modify or understand the underlying code structure.

**Note:** The complete source code of the `ONE_APP` **App**, as assembled and executed throughout this tutorial, is available in the repository under [`src/apps/ONE_APP`](https://github.com/nuremics/nuremics-labs/tree/main/src/apps/ONE_APP).

## Use App

Now that the **App** has been fully implemented and assembled, this new section focuses on its usage from the end-user's perspective.

We will demonstrate how to interact with a ready-to-use **NUREMICS® App**, including how to provide inputs, run the **App**, and retrieve the expected outputs, all without needing to understand its internal structure.

This section assumes the job of the developer is done, and shifts the focus to the operational phase of the **App**.

### Define Working Environment

Before running a **NUREMICS® App**, the operator must first define the working environment in the `if __name__ == "__main__":` section of the **App**.

This setup step specifies two key elements:
- **Working directory (`working_dir`):** The root path where all input/output data, logs, and results will be stored.
- **Study names (`studies`):** A list of identifiers corresponding to the different studies the operator wants to run. Each study will be managed in its own dedicated folder under the working directory.

```python
if __name__ == "__main__":
    
    # ------------------------ #
    # Define working directory #
    # ------------------------ #
    working_dir = Path(os.environ["WORKING_DIR"])

    # -------------- #
    # Define studies #
    # -------------- #
    studies = [
        "Study1",
        "Study2",
    ]

    # --------------- #
    # Run application #
    # --------------- #
    main(
        working_dir=working_dir,
        studies=studies,
    )
```

In this example:
- The `working_dir` is read from the environment variable `WORKING_DIR` previously introduced.
- The `studies` list contains the names of the studies you want to run. You can define as many studies as needed, each representing a self-contained parametric study.

As the **App** is executed, a new folder named after the **App** is automatically created under the specified `working_dir`. This folder acts as the root container for all the execution-related content generated by the **App**.

👤👁️💾
```bash
<working_dir>/
└── ONE_APP/
    └── studies.json
```

📁 **App folder** (`ONE_APP/`)<br>
The name of this folder is derived from the name of the application (`APP_NAME`) as defined during its construction.

📄 **Studies configuration file** (`studies.json`)<br>
This file serves as a centralized configuration hub for each study, allowing the operator to specify which input data remain fixed and which can vary across various experiments.

### Configure Studies

The **NUREMICS®** terminal now provides feedback on the defined studies and halts execution, indicating that the first study `Study1` requires configuration.

👤🔄🖥️
```shell
> STUDIES <

| Study1 |
(X) parameter1 not configured.
(X) parameter2 not configured.
(X) parameter3 not configured.
(X) input1.txt not configured.
(X) input2.json not configured.
(X) input3 not configured.

(X) Please configure file :
> .../ONE_APP/studies.json
```

Let's configure `Study1` by allowing only `parameter1` to vary (by assigning a `true` value in the `studies.json` file), and keeping all other input data fixed (by assigning `false` values in the `studies.json` file) across the study.

📄 `studies.json`
```json
{
    "Study1": {
        "execute": true,
        "user_params": {
            "parameter1": true,
            "parameter2": false,
            "parameter3": false
        },
        "user_paths": {
            "input1.txt": false,
            "input2.json": false,
            "input3": false
        }
    },
    "Study2": {
        "execute": true,
        "user_params": {
            "parameter1": null,
            "parameter2": null,
            "parameter3": null
        },
        "user_paths": {
            "input1.txt": null,
            "input2.json": null,
            "input3": null
        }
    }
}
```

At the next execution of the **App**, the **NUREMICS®** terminal now prompts that `Study1` is properly configured, but halts indicating that the second study `Study2` still requires configuration.

👤🔄🖥️
```shell
> STUDIES <

| Study1 |
(V) parameter1 is variable.
(V) parameter2 is fixed.
(V) parameter3 is fixed.
(V) input1.txt is fixed.
(V) input2.json is fixed.
(V) input3 is fixed.

| Study2 |
(X) parameter1 not configured.
(X) parameter2 not configured.
(X) parameter3 not configured.
(X) input1.txt not configured.
(X) input2.json not configured.
(X) input3 not configured.

(X) Please configure file :
> .../ONE_APP/studies.json
```

Let's this time configure `Study2` by allowing both `parameter3` and `input2.json` to vary, and keeping all other input data fixed across the study.

📄 `studies.json`
```json
{
    "Study1": {
        "execute": true,
        "user_params": {
            "parameter1": true,
            "parameter2": false,
            "parameter3": false
        },
        "user_paths": {
            "input1.txt": false,
            "input2.json": false,
            "input3": false
        }
    },
    "Study2": {
        "execute": true,
        "user_params": {
            "parameter1": false,
            "parameter2": false,
            "parameter3": true
        },
        "user_paths": {
            "input1.txt": false,
            "input2.json": true,
            "input3": false
        }
    }
}
```

At the next execution of the **App**, the **NUREMICS®** terminal now prompts that both `Study1` and `Study2` are properly configured.

👤🔄🖥️
```shell
> STUDIES <

| Study1 |
(V) parameter1 is variable.
(V) parameter2 is fixed.
(V) parameter3 is fixed.
(V) input1.txt is fixed.
(V) input2.json is fixed.
(V) input3 is fixed.

| Study2 |
(V) parameter1 is fixed.
(V) parameter2 is fixed.
(V) parameter3 is variable.
(V) input1.txt is fixed.
(V) input2.json is variable.
(V) input3 is fixed.
```

Going back to the data tree generated within the defined `working_dir`, we can see that a specific folder for each study has been created.

👤👁️💾
```bash
<working_dir>/
└── ONE_APP/
    ├── studies.json
    ├── Study1/
    └── Study2/
```

### Set Input Data

Each study directory within the data tree now contains an initialized input database that must be completed by the operator.

👤👁️💾
```bash
<working_dir>/
└── ONE_APP/
    ├── studies.json
    ├── Study1/
    │   ├── 0_inputs/
    │   ├── inputs.json
    │   └── inputs.csv
    └── Study2/
        ├── 0_inputs/
        ├── inputs.json
        └── inputs.csv
```

This input database contains:
- **`0_inputs`:** This folder must contain the input files and/or folders defined as `"user_paths"`.
- **`inputs.json`:** This file must contain the input parameters defined as _fixed_ `"user_params"`.
- **`inputs.csv`:** This file must contain the input parameters defined as _variable_ `"user_params"`.

At this stage of the **App** execution, the **NUREMICS®** terminal halts by indicating that the _fixed_ input data must be set.

👤🔄🖥️
```shell
> SETTINGS <

| Study1 |
> Common : (X) parameter2 (X) parameter3 (X) input1.txt (X) input2.json (X) input3

(X) Please set inputs :
> .../ONE_APP/Study1/inputs.json
> .../ONE_APP/Study1/0_inputs\input1.txt
> .../ONE_APP/Study1/0_inputs\input2.json
> .../ONE_APP/Study1/0_inputs\input3
```

For the `Study1`, we are speaking about:
- `parameter2` and `parameter3` within the `inputs.json` file.
- `input1.txt`, `input2.json` and `input3` within the `0_inputs` folder.
<br>

📄 `inputs.json`
```json
{
    "parameter2": -9.81,
    "parameter3": 1.0
}
```
<br>

📄 `0_inputs/input1.txt`
```
This is my title
```
<br>

📄 `0_inputs/input2.json`
```json
{
    "h0": 0.5,
    "v0": 15.0,
    "angle": 45.0
}
```
<br>

📄 `0_inputs/input3/solver_config.json`
```json
{
    "timestep": 0.01
}
```
<br>

📄 `0_inputs/input3/display_config.json`
```json
{
    "fps": 60,
    "size": 800
}
```
<br>

All _fixed_ input data have now been completed within the `Study1` input database.

👤👁️💾
```bash
<working_dir>/
└── ONE_APP/
    ├── studies.json
    ├── Study1/
    │   ├── 0_inputs/
    │   │   ├── input1.txt
    │   │   ├── input2.json
    │   │   └── input3/
    │   │       ├── solver_config.json
    │   │       └── display_config.json
    │   ├── inputs.json
    │   └── inputs.csv
    └── Study2/
        ├── 0_inputs/
        ├── inputs.json
        └── inputs.csv
```

At this stage of the **App** execution, the **NUREMICS®** terminal prompts that all _fixed_ input data are now properly set, but halts by indicating that datasets of _variable_ input data must be defined (to conduct various experiments).

👤🔄🖥️
```shell
> SETTINGS <

| Study1 |
> Common : (V) parameter2 (V) parameter3 (V) input1.txt (V) input2.json (V) input3

(X) Please define at least one dataset in file :
> .../ONE_APP/Study1/inputs.csv
```

Let's define three datasets identified as `Test1`, `Test2` and `Test3` within the `input.csv` file.

📄 `inputs.csv`
|  ID   | parameter1 |
|-------|------------|
| Test1 |            |
| Test2 |            |
| Test3 |            |

We can now see that **NUREMICS®** has properly identified the three defined datasets, but is waiting for the _variable_ input data `parameter1` to be set for each of them.

👤🔄🖥️
```shell
 > SETTINGS <

| Study1 |
> Common : (V) parameter2 (V) parameter3 (V) input1.txt (V) input2.json (V) input3
> Test1 : (X) parameter1
> Test2 : (X) parameter1
> Test3 : (X) parameter1

(X) Please set inputs :
> .../ONE_APP/Study1/inputs.csv
```

Let's thus set some values of `parameter1` for each defined dataset within the `input.csv` file.

📄 `inputs.csv`
|  ID   | parameter1 |
|-------|------------|
| Test1 |     3      |
| Test2 |     4      |
| Test3 |     5      |

**NUREMICS®** finally prompts that all input data are properly set for `Study1`.

👤🔄🖥️
```shell
> SETTINGS <

| Study1 |
> Common : (V) parameter2 (V) parameter3 (V) input1.txt (V) input2.json (V) input3
> Test1 : (V) parameter1
> Test2 : (V) parameter1
> Test3 : (V) parameter1
```

The same job must be done to set the input data for `Study2`.

👤👁️💾
```bash
<working_dir>/
└── ONE_APP/
    ├── studies.json
    ├── Study1/
    │   ├── 0_inputs/
    │   │   ├── input1.txt
    │   │   ├── input2.json
    │   │   └── input3/
    │   │       ├── solver_config.json
    │   │       └── display_config.json
    │   ├── inputs.json
    │   └── inputs.csv
    └── Study2/
        ├── 0_inputs/
        │   ├── 0_datasets/
        │   │   ├── Test1/
        │   │   │   └── input2.json
        │   │   ├── Test2/
        │   │   │   └── input2.json
        │   │   └── Test3/
        │   │       └── input2.json
        │   ├── input1.txt
        │   └── input3/
        │       ├── solver_config.json
        │       └── display_config.json
        ├── inputs.json
        └── inputs.csv
```
<br>

Let's here consider the same _fixed_ input data `input1.txt` and `input3` as for the previous study `Study1`.

<br>

📄 `inputs.json`
```json
{
    "parameter1": 4,
    "parameter2": -9.81
}
```
<br>

📄 `inputs.csv`
|  ID   | parameter3 |
|-------|------------|
| Test1 |    1.0     |
| Test2 |    0.1     |
| Test3 |    0.05    |

<br>

📄 `0_inputs/0_datasets/Test1/input2.json`
```json
{
    "h0": 0.5,
    "v0": 15.0,
    "angle": 45.0
}
```
<br>

📄 `0_inputs/0_datasets/Test2/input2.json`
```json
{
    "h0": 0.5,
    "v0": 20.0,
    "angle": 45.0
}
```
<br>

📄 `0_inputs/0_datasets/Test3/input2.json`
```json
{
    "h0": 0.5,
    "v0": 20.0,
    "angle": 60.0
}
```
<br>

**NUREMICS®** is now prompting that all input data are properly set for both `Study1` and `Study2`.

👤🔄🖥️
```shell
> SETTINGS <

| Study1 |
> Common : (V) parameter2 (V) parameter3 (V) input1.txt (V) input2.json (V) input3
> Test1 : (V) parameter1
> Test2 : (V) parameter1
> Test3 : (V) parameter1

| Study2 |
> Common : (V) parameter1 (V) parameter2 (V) input1.txt (V) input3
> Test1 : (V) parameter3 (V) input2.json
> Test2 : (V) parameter3 (V) input2.json
> Test3 : (V) parameter3 (V) input2.json
```

### Get Results

**NUREMICS®** is now ready to run all the defined studies and generate the corresponding results.

👤🔄🖥️
```shell
> RUNNING <

| Study1 | OneProc | Test1 |
> param2 = 3
> param1 = 0.5
> path1 = .../ONE_APP/Study1/0_inputs/input1.txt
>>> START
COMPLETED <<<

| Study1 | OneProc | Test2 |
> param2 = 4
> param1 = 0.5
> path1 = .../ONE_APP/Study1/0_inputs/input1.txt
>>> START
COMPLETED <<<

| Study1 | OneProc | Test3 |
> param2 = 5
> param1 = 0.5
> path1 = .../ONE_APP/Study1/0_inputs/input1.txt
>>> START
COMPLETED <<<

| Study1 | AnotherProc | Test1 |
> param1 = -9.81
> param2 = 1.0
> path1 = .../ONE_APP/Study1/0_inputs/input2.json
> path2 = .../ONE_APP/Study1/0_inputs/input3
> path3 = .../ONE_APP/Study1/1_OneProc/Test1/output1.csv
>>> START
COMPLETED <<<

| Study1 | AnotherProc | Test2 |
> param1 = -9.81
> param2 = 1.0
> path1 = .../ONE_APP/Study1/0_inputs/input2.json
> path2 = .../ONE_APP/Study1/0_inputs/input3
> path3 = .../ONE_APP/Study1/1_OneProc/Test2/output1.csv
>>> START
COMPLETED <<<

| Study1 | AnotherProc | Test3 |
> param1 = -9.81
> param2 = 1.0
> path1 = .../ONE_APP/Study1/0_inputs/input2.json
> path2 = .../ONE_APP/Study1/0_inputs/input3
> path3 = .../ONE_APP/Study1/1_OneProc/Test3/output1.csv
>>> START
COMPLETED <<<

| Study2 | OneProc |
> param2 = 4
> param1 = 0.5
> path1 = .../ONE_APP/Study2/0_inputs/input1.txt
>>> START
COMPLETED <<<

| Study2 | AnotherProc | Test1 |
> param2 = 1.0
> param1 = -9.81
> path2 = .../ONE_APP/Study2/0_inputs/input3
> path1 = .../ONE_APP/Study2/0_inputs/0_datasets/Test1/input2.json
> path3 = .../ONE_APP/Study2/1_OneProc/output1.csv
>>> START
COMPLETED <<<

| Study2 | AnotherProc | Test2 |
> param2 = 0.1
> param1 = -9.81
> path2 = .../ONE_APP/Study2/0_inputs/input3
> path1 = .../ONE_APP/Study2/0_inputs/0_datasets/Test2/input2.json
> path3 = .../ONE_APP/Study2/1_OneProc/output1.csv
>>> START
COMPLETED <<<

| Study2 | AnotherProc | Test3 |
> param2 = 0.05
> param1 = -9.81
> path2 = .../ONE_APP/Study2/0_inputs/input3
> path1 = .../ONE_APP/Study2/0_inputs/0_datasets/Test3/input2.json
> path3 = .../ONE_APP/Study2/1_OneProc/output1.csv
>>> START
COMPLETED <<<
```