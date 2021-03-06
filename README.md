# pypi4u

pypi4u is a python based project that provides a TMCMC and covariance matrix adaptation evolution strategy implementation (CMA-ES) to uncertainty quantification and parameter estimation. The CMA-ES implementation uses the covariance matrix adaptation evolution strategy to determine the maximum of the posterior probability distribution, which is defined as following: 

![equation](http://latex.codecogs.com/gif.latex?p%28hypothesis%7Cdata%2CI%29%20%5Cpropto%20p%28data%7Chypothesis%2CI%29%5Ctimes%20p%28hypothesis%7CI%29)

The TMCMC implementation directly generates samples from the above probability function by using a markov chain. The generated samples can then be used to determine the stochastic mean and variance. The stochastic mean of the multivariate distribution can be equated to the most-likely parameters/estimators that define the trend of the data. 

## Getting Started
The covariance matrix adaptation evolution strategy (CMA-ES) implementation requires python 3.  Furthermore, the following python packages need to be installed: 

* cma 2.5.3 - https://pypi.python.org/pypi/cma
* numpy
* ConfigParser
* matplotlib
* importlib

## How it Works
The following section explains the project's underlying structure and how the provided code can be used to make estimations of the model parameters. This explanation is further supported by a proceeding example, which illustrates how the scripts can be implemented.

### Common Parameters
Both the CMA-ES and TMCMC implementation access a common parameter file, named `common_parameters.par`. The common parameter file, which needs to be filled out by the user, defines the problem and therefore forms the project's foundation. The structure of the common parameter file is depicted below. It consists of three sections; the model, priors and log-likelihood. 

```
[MODEL]
Number of model parameters = 3
model file = model_function.py
data file = data.txt 

[PRIORS]
# Set prior distribution
# prior distributions uniform normal

P1 = normal 4 2
P2 = normal 1 2
P3 = uniform 0 5
error_prior = uniform 0 2

[log-likelihood]
# error either proportional or constant
error = constant
```

**[MODEL]** - In the model section the number of model parameters is to be defined. The model parameters are the number of unknown parameters in the model function. In other words the model parameters are the parameters that are to be predicted. For example if the model function is the following: 

![equation](http://latex.codecogs.com/gif.latex?f%28t%2C%5Ctheta_1%2C%5Ctheta_2%2C%5Ctheta_3%29%3Dt%5Ccdot%5Ctheta_3%5Ccdot%5Ccos%28%5Ctheta_1%5Ccdot%20t%29%20&plus;%20%5Ctheta_2%5Ccdot%5Csin%28t%29) 

The model parameters would be ![equation](http://latex.codecogs.com/gif.latex?%5Ctheta_1%2C%5Ctheta_2%2C%5Ctheta_3) and thus the number of model parameters would be 3. The model file should be set equal to path of the python script that contains the function definition corresponding to the model function. Finally, the data file is the path to the text file that contains a list of input values and corresponding output values (function evaluations with noise).

**[PRIORS]** - In this section the user is able to set the prior probability density functions of the estimators. The prior probability distribution functions can either be normal or uniform. They are assigned by writing to the parameter file P[number of parameter] = [normal] [mean] [variance] or P[number of parameter] = [uniform] [minimum] [maximum]. The error prior defines the prior knowledge available in regards to the noise that corrupts the data. Its definition is identical to that of the parameter priors, just that instead of P[number of parameter], the user must now set error_prior equal to a uniform or normal distribution.

**[log-likelihood]** - In this section the error/noise that corrupts the data can be defined. A constant error means that the data is distorted by a constant term ![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%5Csim%20%5Cmathcal%7BN%7D%280%2C%5C%2C%5Csigma%5E%7B2%7D%29). In the case of a proportional error, the magnitude of the error also depends on *t*, the independent variable, as it is defined as ![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%20%5Ccdot%20t), where ![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%5Csim%20%5Cmathcal%7BN%7D%280%2C%5C%2C%5Csigma%5E%7B2%7D%29). 

### CMA Parameters
Besides setting the common parameters, the user must also define parameters specific to the implementation. The CMA parameters, which are stored in `CMA_parameters.par` file, are the following: 

```
[PARAMETERS]
#defining the parameters for CMA 

bounds = 0 10 #upper and lower bound, the parameters must be within these bounds 
x_0 = 5 5 5 5 #starting point, initial guess for the theta vector (the last entry of the vector corresponds to the guess of the error term)
sigma_0 = 5 #initial standard deviation
```

These specific parameters can be interpreted as following:
* **Bounds** - defines the lower and upper bound of the estimators. The values of all of the estimated parameters are restricted to this bound. The larger the bound the longer it will take for the CMA-ES algorithm to find the maximum of the posterior probability function. 
* **x_0** - this is a vector containing the initial guesses of the estimators. The vector size exceeds the number of model parameters by one. The variance introduced by the noise (![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%5Csim%20%5Cmathcal%7BN%7D%280%2C%5C%2C%5Csigma%5E%7B2%7D%29)) is also an unknown that has to be predicted. It forms the last entry of theta vector. x_0 represents the starting point of the CMA-ES algorithm. Ultimately, the algorithm evolves from this guess towards the most-likely estimators. A rule of thumb is that the initial guesses should be in the middle of bound. If the lower bound is 0 and the upper bound is 10, the x_0 should be 5 5 5 5. 
* **sigma_0** - defines the initial standard deviation used by CMA-ES algorithm when making its initial guesses. 


### TMCMC Parameters

```
[SIMULATION SETTINGS]
pop_size = 10000
bbeta = 0.04
tol_COV = 1
BURN_IN = 20
# max_stages = 100
#seed = -1

[optimization settings]
# OPTIONAL
#max_stages

#Either here or in an additional default settings file
[DEFAULT]
max_stages = 10000
seed = -1
MaxIter = 1000
```

### Model Function
The model function needs to be defined by the user. It is a function that takes two arguments, an estimator vector of a given size (size is defined in common parameters) and *t*, and returns a float. For example: 

```
import math

def model_function(theta, time): #evaluates my model function for a given theta and time
	return time*theta[2]*math.cos(theta[0]*time) + theta[1]*math.sin(time)
```


### Data File
The user needs to append a data file. This data file should be a text file that contains two columns, delimited by a space. The first column should be the value of the independent variable [*t*], while the second column should be corresponding function evaluation/measurement [*function evaluation*]. 

### Executing the Code
After having filled in the parameter files, the estimators for the model parameters are simply obtained by either running `CMA_implementation.py` or `TMCMC_implementation.py`. On execution a text file named `CMA_estimators.txt` or `TMCMC_estimators.txt` will be created, in which the values of the estimators are stored. The last estimator in the file corresponds to the error estimator. It estimates the variance of the noise, within the data set. 

## Example Problem - DEMO 

### Generation of Synthetic Data
Synthetic data was generated from a predefined model function:

![equation](http://latex.codecogs.com/gif.latex?f%28t%2C%5Ctheta_1%2C%5Ctheta_2%2C%5Ctheta_3%29%3Dt%5Ccdot%5Ctheta_3%5Ccdot%5Ccos%28%5Ctheta_1%5Ccdot%20t%29%20&plus;%20%5Ctheta_2%5Ccdot%5Csin%28t%29) 

The model parameters were set equal to ![equation](http://latex.codecogs.com/gif.latex?%5Ctheta_1%20%3D%204%2C%20%5Ctheta_2%3D1%2C%20%5Ctheta_3%3D2). The function was then evaluated for ![equation](http://latex.codecogs.com/gif.latex?t%20%3D%20%5B0.2%2C%200.4%2C%20%5Chdots%2C%204.0%5D). Additionally, random noise is introduced by simply adding epsilon to the function evaluations (constant error). The sum of the terms forms 

![equation](http://latex.codecogs.com/gif.latex?y_i%20%3D%20f%28t_i%2C%5Ctheta_1%2C%5Ctheta_2%2C%5Ctheta_3%29&plus;%5Cvarepsilon)

where epsilon equates to ![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%20%5Csim%20%5Cmathcal%7BN%7D%28%5C0%2C1%29)

Consequently, all obtained function evaluations are independently and identically distributed, following a normal distribution with a variance of one. The synthetic data is stored in a text document `data.txt`, which lists the input value *t* and the corresponding function value *f*. Both approaches use the synthetic data and the function definition *f* to approximate the values of the thetas and epsilon. 

### Common Parameters
```
[MODEL]
Number of model parameters = 3
model file = model_function.py
data file = data.txt 

[PRIORS]
# Set prior distribution
# prior distributions uniform normal

P1 = normal 4 2
P2 = normal 1 2
P3 = uniform 0 5
error_prior = uniform 0 2

[log-likelihood]
# error either proportional or constant
error = constant
```
**[MODEL]** - The model function consists of three parameters; therefore the number of model parameters was set to three. Additionally, the paths to the python model function and to the data file are given. 

**[PRIORS]** - In this exemplary case, the prior for the first parameter was taken to be a normal probability distribution with a mean of 4 and a variance of 2. The prior of the second parameter is also a normal probability distribution, but with a mean of 1 and a variance of 2. The third prior was set to a uniform probability distribution with a minimum of 0 and maximum of 5. Finally, the error prior was defined to be a uniform distribution with a minimum of 0 and maximum of 2. 

**[log-likelihood]** - The synthetic data was produced by corrupting the function evaluations with constant noise, which originated from a normal distribution with a mean of 0 and a variance of 1 (![equation](http://latex.codecogs.com/gif.latex?%5Cvarepsilon%20%5Csim%20%5Cmathcal%7BN%7D%28%5C0%2C1%29)). Therefore, the error is set equal to a constant in the log-likelihood section of the common parameters. 

### Model Function - Python Function 
The model function is defined as following: 

![equation](http://latex.codecogs.com/gif.latex?f%28t%2C%5Ctheta_1%2C%5Ctheta_2%2C%5Ctheta_3%29%3Dt%5Ccdot%5Ctheta_3%5Ccdot%5Ccos%28%5Ctheta_1%5Ccdot%20t%29%20&plus;%20%5Ctheta_2%5Ccdot%5Csin%28t%29) 

Therefore, the first argument of the function, the theta vector, needs to be a vector of size three, as there are three model parameters. The resulting function definition is as following: 

```
import math

def model_function(theta, time): #evaluates my model function for a given theta and time
	return time*theta[2]*math.cos(theta[0]*time) + theta[1]*math.sin(time)
```

Both the CMA-ES and the TMCMC implementation call this python function.  
### CMA-ES Implementation
To be able to implement the CMA-ES algorithm the CMA parameters must still be defined.  

```
[PARAMETERS]
#defining the parameters for CMA 

bounds = 0 10 #upper and lower bound, the parameters must be within these bounds 
x_0 = 5 5 5 5 #starting point, initial guess for the theta vector (the last entry of the vector corresponds to the guess of the error term)
sigma_0 = 5 #initial standard deviation
```

In this example all parameters lie within the bound [0,10]. Furthermore, the rule of thumb is applied to obtain an initial starting guess for the theta vector. Finally, the initial standard deviation of the CMA-ES alogrithm was defined to be 5. 








