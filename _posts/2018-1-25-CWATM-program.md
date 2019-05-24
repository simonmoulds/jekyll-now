---
layout: post
title: The Community Water Model
---

One of the many interesting talks I attended at AGU 2018 was an introduction to the [IIASA Community Water Model](https://cwatm.github.io/) given by Peter Burek. The model is implemented in Python and is free and open source under the terms of the GNU General Public Licence (version 3). CWATM follows an object oriented design which employs the PCRaster dynamic model concept. I have no experience with PCRaster and most of my knowledge of OOP comes from R, which is very different to OOP in Python (and indeed most OOP languages). Therefore in this post I attempt to give an overview of the program structure. 

# Program structure

The main program is contained in `cwatm.py`. When a Python file is run as the main program from the command line the Python interpreter sets a number of special variables. One of these variables, `__name__`, is set to equal `__main__`, which means that the main script is interpreted by Python as a module with the name `__main__`. On the other hand, if the file is being imported from another module the `__name__` variable will be set to the module's name (https://stackoverflow.com/questions/419163/what-does-if-name-main-do). Hence, as is usual for Python programs, the main program cwatm.py contains the control structure `if __name__ == __main__` and a series of commands which parse command line arguments and run the model. The statement is not a function and hence the variables defined therein are global variables to which all scopes have access (https://stackoverflow.com/questions/4775579/main-and-scoping-in-python). 

The main program file defines class `CWATModel` and a function `CWATMexe()` to execute the model. Function `CWATMexe()` first parses the configuration file with the command

```python
parse_configuration(settings)
```

which populates the global lists `binding` and `option` with the model settings. Next, `CWATMexe()` reads the metadata for NetCDF output files:

```python
read_metanetcdf(cbinding('metaNetcdfFile'), 'metaNetcdfFile')
```

populating the global dictionary `metaNetcdfFile` with a list of variables to be written to file and the corresponding NetCDF metadata (i.e. author, long\_name, standard\_name, title, unit). The global variable `dateVar`, which is used to keep track of time during the simulation, is then initialized:

```python
checkifDate('StepStart','StepEnd','SpinUp')
```

where 'StepStart', 'StepEnd' and 'SpinUp' are variables in the global list `binding`. This function populates the global object `dateVar` with information about the simulation period and the first time point. The `dateVar` object, which is updated before every subsequent time step, is the means by which the program keeps track of time. The program proceeds to create an instance of class `CWATModel`, a subclass of the `CWATModel_ini` and `CWATModel_dyn` base classes (Python allows multiple inheritance), which are defined in files `cwatm_initial.py` and `cwatm_dynamic.py`, respectively. Class `CWATModel_ini`, which inherits from PCraster class `DynamicModel`, includes the `__init__` method which initializes the `CWATModel` object. Attributes of `CWATModel_ini` include the various modules in directory `hydrological_modules`. Each hydrological module consists of a class to represent an aspect of the hydrological model, with each class including two methods: `initial()` and `dynamic()`. After setting these attributes the modules are initialized by calling their `initial()` method, e.g.

```python
# assign the module to a class attribute
self.waterbalance_module = waterbalance(self)

# initialize the module
self.waterbalance_module.initial()
```

Class `CWATModel_dyn` contains one method, `dynamic()`, which calls the `dynamic()` method of each hydrological module initialized in class `CWATModel_ini`. The resulting `CWATModel` object, which complies with the PCRaster dynamic model through its inheritance of the DynamicModel class and its implementation of a `dynamic()` method, is supplied to the constructor function of the PCRaster class `DynamicFramework`. Two additional input arguments are required: `firstTimeStep` and `lastTimeStep`, which are retrieved from the `dateVar` object. The `__init__` method of DynamicFramework assigns arguments userModel and firstTimeStep to attributes `_userModel` and `_d_firstTimeStep` respectively, while argument lastTimeStep is passed to the `DynamicModel._setNrTimeSteps()`, which updates DynamicModel attribute `_d_nrTimeSteps`.

The time loop itself is implemented in `DynamicFramework.run()`. The method first calls `DynamicFramework._runInitial()`, which in turn calls `userModel().initial()` (defined in DynamicModel), and then `DynamicFramework._runDynamic()`. This method sets a variable 'step' to equal the first time step (`self._userModel().firstTimeStep()`) and then initiates a time loop. At each time step the DynamicModel attribute `currentStep` is updated with the `_setCurrentTimeStep` setter (`currentStep` is retrieved with the `currentTimeStep` getter). An annotated version of the `_runDynamic` method is provided below:

```python
def _runDynamic(self):
    
    # set DynamicModel attribute inDynamic to True
    self._userModel()._setInDynamic(True)
    
    # get DynamicModel attribute _d_firstTimeStep
    step = self._userModel().firstTimeStep()
    
    # loop over total number of time steps
    while step <= self._userModel().nrTimeSteps():
        
        self._incrementIndentLevel()
        self._atStartOfTimeStep(step)
        self._userModel()._setCurrentTimeStep(step)

        if hasattr(self._userModel(), 'dynamic')
            self.incrementIndentLevel()
            self._traceIn("dynamic")
            
            # here the model calls the dynamic() method
            # in CWATM this is defined in CWATModel_dyn
            self._userModel().dynamic()
            self._traceOut("dynamic")
            self._decrementIndentLevel()
            
        self._timeStepFinished()
        self._decrementIndentLevel()
        step += 1

    self._userModel()._setInDynamic(False)
```

Method `CWATModel.dynamic()` (defined in base class `CWATModel_dyn`), which is called at each time point in `DynamicFramework._runDynamic()`, calls function `timestep_dynamic()` (defined in management\_modules/timestep.py) which updates global variable `dateVar`. When `dateVar` is initialized the `curr` key is set to zero and `currDate` equals the simulation start date (key `dateStart`). In this way the `dynamic()` method of the hydrological modules can retrieve information about the current time point and load input data accordingly. 

# Modules

## Management modules
The management\_modules directory contains a number of files which define various functions to coordinate the model run. This includes functions to configure the model (configuration.py; parse\_configuration), perform runtime checks (checks.py), manipulate data (data\_handling.py), define global variables and check the system requirements are met (globals.py), provide runtime information (messages.py), write output (output.py), handle timestepping (timestep.py) and augment some PCRaster functionality (improvepcraster.py, replace\_pcr.py). 

### configuration.py

Function parse_configuration (configuration.py) is essentially a wrapper around ConfigParser (https://docs.python.org/2/library/configparser.html), but includes an extension ExtParser which conveniently enables the user to use file handles to specify the paths of input variables. This means that users only need to change the handles rather than every single filepath. 

## Hydrological modules

Subdirectory hydrological_modules contains various files which each define a class to represent specific hydrological processes (e.g. capillary rise, evaporation). Each class definition includes 'initial' and 'dynamic' methods. 

## References

