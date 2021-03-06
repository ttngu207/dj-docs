## Imaging:  Web GUI

1. Copy files to a file server
2. Open the [imaging web GUI](http://2p.neuroballs.net:5000/) and log in 
    - Go to _BaseFolder_ -> _Add BaseFolder_
3. Fill out the details in the mask on that website. Make sure you get the session logic **_combined_** right. It determines how (meta)sessions are created in the database. This only has relevance for data acquired with scanimage (miniscope, ...). For femtonics recordings this function is not implemented and has to remain on "no" (not combined). **Explanation**: 


    - Scanimage creates filenames like `90218-openfield1m_00001_00001.tif`, where `90218-openfield1m` is the name that the experimenter chose within scanimage for that recording. It makes sense to create a name consisting of the mLIMS animal ID and some keyword that identifies what kind of session it was (openfield, etc.). There is no need to specify the date since this information is saved within the header of recorded tif files. 
    ![Miniscope session folder screenshot](../_static/imaging/scanimage_basefolder.PNG) <br />
    Scanimage then has two counters: `_00001_00001.tif`. The first counter refers to how many times a user clicked abort or recorded a full session and _**then started again under the same name**_. This can happen, for example, if the pre-set number of acquired frames was estimated too low and the experimenter wishes to extend the current session. Or if there was a small problem with the recording (the animal twisted itself, ...) and that current recording had to be interrupted briefly. This has nothing to do with the _combined_ logic and these interrupts are considered to be insignificant and have no relevance for downstream processing. So no matter how many times the experimenter clicked stop and started again, this will all be stitched together and count as _the same_ **Session**. The second counter refers to the actual file number if tif splitting is activated within scanimage, e.g. if the experimenter specified that a maximum of 2000 frames should be saved within one file. 
    - When the experimenter changes the name within scanimage (for example records a new head fixed session after an open field session), this will count as a new **Session**, regardless of the _combined_ logic. 
    - The _combined_ logic determines whether multiple sessions within one folder should be analyzed together. If **_combined = no_**, then each session found in that folder will get its own `Session` and `MetaSession` entries (all distinct from each other). If **_combined = yes_**, then each session found in that folder will get its own `Session` entry, but they will be grouped under the same `MetaSession` entry. If sessions are not combined, they will run through the analysis separately and individual output will be generated for each session. If they are however in the same `MetaSession`, then only one, **combined** output will be generated (the imaging analysis is linked to the MetaSession level). Here is a diagram that I hope helps to clarify this further: ![Session combined logic](https://github.com/kavli-ntnu/dj-moser-imaging/blob/master/wiki_files/session%20combined%20logic.jpg)
  
4. Lean back
5. Check the `Sessions` subpage in the web GUI: Your newly added sessions should pop up there. If you have deep lab cut (DLC) data, you have to associate a `Model | ProcessingMethod` combination on a per session basis. Valid selections will be shown to you when you click on the blue `DLC` button. After that (or if you do not have any DLC data), you can click on the red `None` icon to fill out your session type and apparatus info. This will make sure that the correct session and apparatus information is found by downstream processes. If the session was an object session, you can proceed to [Enter object locations Napari.ipynb](https://github.com/kavli-ntnu/dj-moser-imaging/blob/master/Helper_notebooks/Enter%20object%20locations%20Napari.ipynb), which lets you specify details (location, type) of objects in the arena. 
6. Check whether the imaging analysis has run through (only suite2p at the moment). This information can be retrieved from the imaging web GUI (under _Suite2p_ -> _Finished Suite2p Jobs_). 
7. Curate the imaging analysis results (Suite2p GUI)
8. Once you are happy with the results, re-add the session BaseFolder to the web GUI. Use **the same information** as when you added the BaseFolder the first time (take care of suite2p options and combined status). There is a convenient way of doing that: Just click the button "Add" behind the session under _Suite2p_ -> _Finished Suite2p Jobs_ in the web GUI.
9. Datajoint will re-ingest the BaseFolder, but only add information that has been missing before (the imaging analysis (suite2p) output. 
10. Lean back and watch things being calculated (web GUI: _Jobs_ -> _Jobs Imaging_)
11. Go through the notebooks to make sense of your analysis results or use the [Session Viewer GUI](https://github.com/kavli-ntnu/dj-moser-imaging/tree/master/viewer) to inspect the results (the GUI is work in progress, so don't expect too much right now).

### What happens if I forgot to copy over a file? 
Just add the BaseFolder entry again. The ingest routines will recognize what is new and only add that. 

### What happens if I don't like this or that cell from the suite2p output? 
If you re-ingested the imaging analysis output once and then change something from within the suite2p GUI, you have to add the session BaseFolder again for the imaging pipeline to be notified of that change. Once re-added, the ingest routines will detect a mismatch between cell IDs saved in datajoint and the suite2p output and all datajoint results will be deleted for that session (those that derive from the imaging analysis) and re-calculated.

### If things fail: 
- Check the [imaging web GUI](http://2p.neuroballs.net:5000) (_Jobs_ -> _Jobs Imaging_)
- If there was an error in the `MakeDatasetsSessions` (so during the basic ingest), make sure the computer that runs the `MakeDatasetsSessions` job knows about the file server that the raw data was saved under (that it is listed in `network_drives.cfg`)
- Ask Horst or Simon on Teams or via email. 
