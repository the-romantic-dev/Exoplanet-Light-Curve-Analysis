# Exoplanet Light Curve Analysis

A python 3 package for modeling exoplanet light curves. The transit function is based on the analytic expressions of Mandel and Agol 2002.

- Simple transit generator
- Easily create noisy datasets
- Parameter optimization and uncertainty estimation (powered by Scipy)

### For posterior inference in a Bayesian framework please see the [nested](https://github.com/pearsonkyle/Exoplanet-Light-Curve-Analysis/tree/nested) branch

![ELCA](https://github.com/pearsonkyle/Exoplanet-Light-Curve-Analysis/blob/master/Lightcurve%20Fit.png "Light Curve Modeling")



## Running the package
```python
from ELCA import lc_fitter, transit
import numpy as np

if __name__ == "__main__":

    t = np.linspace(0.85,1.05,200)

    init = { 'rp':0.06, 'ar':14.07,       # Rp/Rs, a/Rs
             'per':3.336817, 'inc':88.75, # Period (days), Inclination
             'u1': 0.3, 'u2': 0,          # limb darkening (linear, quadratic)
             'ecc':0, 'ome':0,            # Eccentricity, Arg of periastron
             'a0':1, 'a1':0,              # Airmass extinction terms
             'a2':0, 'tm':0.95 }          # tm = Mid Transit time (Days)

    # only report params with bounds, all others will be fixed to initial value
    mybounds = {
              'rp':[0,1],
              'tm':[min(t),max(t)],
              'a0':[-np.inf,np.inf],
              'a1':[-np.inf,np.inf]
              }


    # GENERATE NOISY DATA
    data = transit(time=t, values=init) + np.random.normal(0, 2e-4, len(t))
    dataerr = np.random.normal(300e-6, 50e-6, len(t))

    myfit = lc_fitter(t,data,
                        dataerr=dataerr,
                        init= init,
                        bounds= mybounds,
                        )

    for k in myfit.data['freekeys']:
        print( '{}: {:.6f} +- {:.6f}'.format(k,myfit.data['LS']['parameters'][k],myfit.data['LS']['errors'][k]) )

    myfit.plot_results(show=True,phase=True)

    # explore the output of the fitting
    print( myfit.data['LS'].keys() )
```

## Output

```python 
myfit = {
    'LS': {
        'res': ndarray,         # Optimize Result from scipy.optimize.least_squares fit
        'finalmodel': ndarray,  # best fit model of light curve (transit+detrending model)
        'residuals': ndarray,   # residual from light curve fit (data-finalmodel)
        'transit': ndarray,     # just the transit model with no system trend
        'phase': ndarray,       # lightcurve phase calculation based on fit mid transit
        'parameters':{             
            'rp': float, 'ar': float,   # Rp/Rs, a/Rs
            'per': float, 'inc': float, # Period (days), Inclination
            'u1': float, 'u2':  float,  # limb darkening (linear, quadratic)
            'ecc': float, 'ome': float, # Eccentricity, Arg of periastron
            'tm': float                 # tm = Mid Transit time (Days)
            },
        'errors':{
            # same format as parameters
            # uncertainty estimate on parameters 
        }               
    }
    
    # For posterior parameter distributions please see the "nested" branch of this repo
}
```


## Set up and install from scratch

Clone the git repo
```
cd $HOME
git clone https://github.com/pearsonkyle/Exoplanet-Light-Curve-Analysis.git
```
Rename for simplicity later
```
mv Exoplanet-Light-Curve-Analysis ELCA
```
Compile the C code. Python will link to this file later
```
cd ELCA/util_lib
chmod +x compile
./compile
```
Create two PATH variables so that this code can be accessed from anywhere on your computer. This keeps all of the codes in one location for easy updating and referencing.
```
cd $HOME
gedit .bashrc
```
**add the two lines below to your .bash_profile or .bashrc**
```
export PYTHONPATH=$HOME/ELCA:$PYTHONPATH
export ELCA_PATH=$HOME/ELCA/util_lib
```
Update your .bashrc file after adding those two lines
```
source .bashrc
```


## Citation 
If you use these algorithms please cite the article below

[Pearson K. A., et al., 2019, AJ, 157, 21](http://iopscience.iop.org/article/10.3847/1538-3881/aaf1ae/meta)

Here is an example bibtex
```
@article{Pearson2019,
  author={Kyle A. Pearson and Caitlin A. Griffith and Robert T. Zellem and Tommi T. Koskinen and Gael M. Roudier},
  title={Ground-based Spectroscopy of the Exoplanet XO-2b Using a Systematic Wavelength Calibration},
  journal={The Astronomical Journal},
  volume={157},
  number={1},
  pages={21},
  url={http://stacks.iop.org/1538-3881/157/i=1/a=21},
  year={2019}
}
```
