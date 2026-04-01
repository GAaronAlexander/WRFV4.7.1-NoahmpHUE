### Compiling WRF
Because of the number of new variables that are added, SSIB and CLM cannot be ran from the same WRF executable as Noah-MP Mosaic or HUE. When compiling, pass the following option to compile WRF with Noah-MP Mosaic
``` ./configure hue```

This will "hide" certain variables in the WRF workflow that will for sure not be used by noah-MP mosaic and HUE. 

If you would like to CLM or SSIB, either download a new version of WRF 4.7.1 without this code changes or run configure without the "hue" keyword.
### Pre Processing Steps
- After running geogrid and wanting to use **NLCD** landcovers run MapNewLUGeogrid script.
	- If there is a glacier landcover, this has to be removed. the Noah-MP glacier code is very prone to leaking water, and cannot be averaged together with other landcovers at this time. 
- proceed with ungrib and metgrid as normal. 

### Steps for Running Real  and WRF

The only change that is needed is a linking of the  **MPTABLE.TBL** for use of **NLCD** data. If you are not running with NLCD, then just ignore this step
``` 
mv MPTABLE.TBL MPTABLE.Original.TBL
ln -s /Location/To/WRF/Repo/ExtraMosaicSteps/MPTABLE.50LandCats.TBL ./MPTABLE.TBL
```


The following Namelist options need to be active: 
&physics
``` num_land_cat = n ``` -> This should be n=50 now for **NLCD**
``` sf_surface_mosaic = 1 ``` -> Turns on a flag to activate mosaicking  
``` mosaic_cat = g ``` -> g is the number of landcovers to mosaic over. I typically use 4 - 8. CANNOT be changed for different domains

&physics
``` opt_mos = 1 ``` -> Turns on Noah-MP Mosaicking 
``` opt_hue = 0 ``` -> This turns off Noah-MP HUE

My suggested set-up for Running Noah-MP Mosaicking for physics is: 
```
dveg                                = 4,
opt_crs                             = 1,
opt_sfc                             = 1,
opt_btr                             = 2,
opt_run                             = 3,
opt_frz                             = 1,
opt_inf                             = 1,
opt_rad                             = 3,
opt_alb                             = 2,
opt_snf                             = 1,
opt_tbot                            = 2,
opt_stc                             = 1,
opt_gla                             = 1,
opt_rsf                             = 1,
opt_soil                            = 1,
opt_irr                             = 0,
opt_crop                            = 0,
```

After that, real.exe can be ran and wrf.exe as normally

### Running WRF

### Notes:
**Under no circumstances should you turn on groundwater schemes or urban schemes at this point**
- Groundwater: There are groundwater options in noah-mp but they require different inputs that noah-MP mosaicking is not ready to handle. Specifically option 5 of runoff will immediately break the model 
- Urban Scheme: The Single Layer Urban Canyon model can be ran BUT it can give some weird results because the SLUCM is itself a mosaicking scheme (so you are mosaicking within a mosaic). If interested in running this, contact gaalexander3@wisc.edu for ways to fix that. 
- BEP+BEM: Under development to run with mosaicking!