# THOR-EAE
To download our dataset, visit the link below:

```
wget 8.137.38.62:8080/data/THOR-EAE.tar.gz
```

## Structure of dataset THOR-EAE

Our dataset contains egocentric images, explanation text, ROIs, metadata.

datasets/FloorPlan x:

	-images:
 
	    |LGN_data/bypass
     
	    |LGN_data/move
     
	    |LGN_data/pickup
     
	    |LGN_data/stepover
     
	    |LGN_data/try_bypass
     
	    |LGN_data/stop
     
	-explanations:
 
	    |LGN_data/cap
     
	    |LGN_data/gpt_cap
     
	-ROIs:
 
	    |LGN_data/rois
	-metadata:
	    |FloorPlan x.json
