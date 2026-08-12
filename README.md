# Xshooter-Easy
A set of scripts meant to "easily" reduce scientific data from the XSHOOTER instrument into usable spectra.

# README:

Version 1, May 2025.

Hello and welcome to the "Xshooter Easy" data reduction tool!
I am Gian Michele Bravo and I learned how to reduce Xshooter data through trial and error.
This tool is meant for you to reduce Xshooter data with less error and more useful science.


## QUICK START:

### Prerequisites:
1) A bash interpreter
2) A python3 interpreter
3) The astropy python package
4) The numpy python package
5) The matplotlib.pyplot python package (for plotting)
3) The common pipeline library for eso recipes
4) The esorex recipe runner
5) The xshooter pipeline recipes

### Execution:
1) Take all the necessary input .fits files and put them in the "Input" folder.
2) Open the terminal in the "Xshooter Easy" directory
2) Run the command
bash xsh_all_recipes.sh
3) Wait for it to finish. This step should take between minutes to tens of minutes. So far, this data reduction has never completed without giving warnings, but it has completed without giving errors.
4) If everything works, you will be able to plot the resulting spectrum using either of the commands
python3 plot_1d.py
python3 plot_2d.py
Furthermore, the output files from the data reduction will be in the "Output" folder.


## POSSIBLE PROBLEMS:

In case something breaks, here are a few problems that the Xshooter Easy Tool might have:

1) For all file paths, the Xshooter Easy tool uses the Linux file path format, ./FOLDER/FOLDER/.../FILENAME.TYPE
Windows operating systems use the symbol \ (backslash) instead of / (slash). This will cause an error. The solution is to replace all file paths containing /. These paths are found in the following scripts:
	xsh_all_recipes.sh
	input_fits_sorter.py
	output_fits_sorter.py
	plot_1d.py
	plot_2d.py

2) While this tool was developed, a common problem was caused by the program misidentifying what a given data file was. For example, the script would erroneously give a flat frame to the xsh_mdark recipe, which uses dark frames. The Xshooter Easy tool tries to recognise which file is which based on information in the file's header, which is analyized through a series of if strings. The functions meant to recognise files are found in the following files:
	xsh_conditions_NIR.py
	xsh_conditions_VIS.py
	xsh_conditions_UVB.py
Depending on what input data the tool is given, the conditions meant to recognise a given data file might be insufficient, and the tool might misinterpret which file is what. This would produce an error in the recipe running step of the tool. To solve this, it is necessary to modify the condition functions themselves. 

3) Finaly, the Xshooter Easy tool reduces data in the slit-stare mode of the telescope, during which the telescope points straight at the target system and its light passes through a thin slits at the start of each instrument arm (explained in the next section). This is not the only observing mode that XSHOOTER can use. In the slit-offset mode, the telescope still uses slits, but it is pointing at an offset compared to the target object. In the nodding mode, the telescope moves up and down over the course of the observation to distribute the incoming light across more pixels and avoiding pixel saturation. In the IFU mode, the telescope uses a so called Integral Field Unit to capture the light across a larger area of the sky than in slit mode, and then projects that light onto the CCD sensors. The Xshooter Easy tool is not built for and has not been tested for these modes. Making the tool work for those requires, at best, changing what reduction recipe is used in the
	xsh_all_recipes_NIR.sh
	xsh_all_recipes_VIS.sh
	xsh_all_recipes_UVB.sh
scripts. At worst, this requires additional scripts to produce the data products that the other observing modes require for data reduction.

## UNDERSTANDING WHAT IS SUPPOSED TO HAPPEN:

It appears that you're out of luck, and you will have to learn what you're actually doing, so that you can understand what you are doing wrong.
In this case, here is an explanation of how Xshooter data reduction works, as well as information as to where you can find more resources. I will try to keep this as clear and concise as possible. I will put particular focus on information that I had difficulty finding on the the information manuals that already exist, and had to learn myself through trial and error.

### The Xshooter instrument:

The Xshooter instrument is an amalgamation of mirrors, echelle gratings, slits, shutters, lenses, beam splitters and CCDs. The point of the instrument is to take all (or at least the majority) of the light that a normal star emits and that passes through the atmosphere, and give the spectra for that light. In practice, this gives a spectra from the ultraviolet to the near infrared. It uses the echelle gratings to split the light along its wavelengths, then projects it onto the CCDs, where the position of the incoming photons corresponds to their wavelength.

This is where we enocunter instrument arms. When the instrument was built, there didn't exists any one CCD that was capable at detecting photons efficiently at both ultraviolet, visible and near-infrared wavelengths. As such, the instrument splits the light onto three different CCDs usin beam splitters, each one optimized to detect photons at its intended wavelength. These three CCDs and their corresponding optics are called the "arms" of the instrument. They are reffered to as the UVB arm, which covers ultraviolet and blue light, the VIS arm, which covers from green light to approximately 1000nm, and the NIR arm, which covers from 1000 nm to around 2400 nm.

The data produced by the instrument are scans of the three CCDs after pointing the telescope with the instrument at the object you are interested of. The xshooter pipeline is meant to take those scans and give you the actual spectra of whatever you pointed at.

Finally, the instrument has several modes they can be used. These relate to the way the instrument is pointing at your objects, and wheter it is looking through a slit or through an integral field unit, or IFU. This is a piece of material that diffuses the light so that the instrument loses spatial resolution and is instead projected uniformly on the CCD, only getting spread by wavelength). Secondly, it can either points straight at an object, it can point at it with some offset, and it can "nod," moving the telescope so that the object moves up and down the slit.
In total, this allows for 5 instrument modes:
slit_stare
slit_offset
slit_nodding
ifu_stare
ifu_offset

The "Xshooter Easy" tool reduces observations taken in the slit_stare mode.

### The pipeline:

The data reduction is divided in different so called recipes. Each recipe is a single script which takes a series of files, the information about what those files are and eventual user-added parameters. From that, it produces new, useful files. As it turns out, all of the input and output files used by xshooter recipes are of the .fits file format. This can be 2D CCD scans as well as data tables.

Running each recipe is done with terminal commands (when using esorex). An example for such a command is 
esorex xsh_mflat xsh_mflat.sof
Translated to words, this means "use esorex to run the xsh_mflat recipe using the series of files (sof) given by the text file xsh_mflat.sof"
The series of files is a necessary part of running each recipe. In practice, it is a text file containing the name of all files used by the recipe, as well as what they are. As an example, for the xsh_mflat recipe in the VIS arm, the .sof file might look as folllowing:

M.XSHOOTER.2019-04-04T10:02:33.076.fits	BP_MAP_RP_VIS
M.XSHOOTER.2019-04-04T10:19:47.773.fits	SPECTRAL_FORMAT_TAB_VIS
MASTER_BIAS_VIS.fits			MASTER_BIAS_VIS
ORDER_TAB_CENTR_VIS.fits		ORDER_TAB_CENTR_VIS
XSHOO.2024-10-03T13:11:06.102.fits	FLAT_SLIT_VIS
XSHOO.2024-10-03T13:11:50.245.fits	FLAT_SLIT_VIS
XSHOO.2024-10-03T13:12:34.539.fits	FLAT_SLIT_VIS
XSHOO.2024-10-03T13:13:18.643.fits	FLAT_SLIT_VIS
XSHOO.2024-10-03T13:14:03.676.fits	FLAT_SLIT_VIS
XSHOO.2024-10-04T11:34:22.186.fits	FLAT_SLIT_VIS
XSHOO.2024-10-04T11:36:01.854.fits	FLAT_SLIT_VIS
XSHOO.2024-10-04T11:37:41.743.fits	FLAT_SLIT_VIS

This is the .sof file in its entirety. The first row contains the file name of a map detailing bad pixels on the CCD, marked with the so called "tag" BP_MAP_RP_VIS. It contains a table that, for each arm, gives the wavelength that the arm can achieve. This file is tagged with SPECTRAL_FORMAT_TAB_VIS. BP_MAP_RP_VIS and SPECTRAL_FORMAT_TAB_VIS are so called "static calibration data," meaning that you can dowload them directly from the ESO archives without the need to produce them yourself. The next row contains is MASTER_BIAS_VIS.fits. This file is actually created by one previous recipe, the xsh_mbias recipe, and as such doesn't follow the naming conventions of the other files. Similarly, ORDER_TAB_CENTR_VIS.fits on the next row is created by the xsh_orderpos recipe. Lastly, there are multiple rows with the tag FLAT_SLIT_VIS. These are flat frames, taken when the telescope is pointing at a flat lamp. Multiple different files are included for the recipe to be able to average out noise.


### The Xshooter Easy tool:

Here's the good news: the "Xshooter Easy" tool is supposed sort your raw data, produce all the .sof files and run all the recipes needed for you. All of this shoud happen automatically when running the xsh_all_recipes.sh. Here's the bad news: if you're reading this, it is likely that something didn't work.

The first step in reducing the data is to sort the input files in appropriate folders. This is done by the input_fits_sorter.py script. It sort input files according to what instrument arm they are useful for.
At this point, it is worth to mention how you can find the instrument arm of a given .fits file. This information is found in the header, under the header key "HIERARCH ESO SEQ ARM".

Xshooter Easy finds the correct files to use for a given recipe simply by looking through all files available, testing wheter each file corresponds to a specific tag that a recipe needs, and if this condition is met, writing it down. The conditions can be found in the xsh_conditions_NIR.py, xsh_conditions_VIS.py and xsh_conditions_UVB.py python scripts. These conditions determine whether a file corresponds to a given tag or not based on information in the file's header. A header is simply a series of keywords with corresponding value and comments. You can view a header yourself simply using the ds9 .fits viewer, or with the following python3 commands:
'''import astropy.table
dat = astropy.io.fits.open('M.XSHOOTER.2019-04-04T09:51:11.626.fits')
header = dat[0].header
header'''
Of course, you can replace the file M.XSHOOTER.2019-04-04T09:51:11.626.fits with whatever .fits file you want.
Calling a header key can also be done simply, and is done throughout the Xshooter Easy tool. Another important header key, which is found for all the non-raw frames (in other words, static calibration data and processed frames) is HIERARCH ESO PRO CATG. For many recipes, this key contains the tag that the file corresponds to. As an example, following the previous code:
>>> header['HIERARCH ESO PRO CATG']
'BP_MAP_RP_NIR'
This means that the people who made the static calibration file M.XSHOOTER.2019-04-04T09:51:11.626.fits helpfully added a line in it that said this file corresponds to a map of bad pixels on the CCD for near-infrared.

The Xshooter Easy tool uses this and other keys to identify correct files and write them in .sof files, and then feeds those .sof files to the recipes. If that part of the program works, you should be able to see that these files have been created in the 'NIR', 'VIS' and/or 'UVB' folders.

When the Xshooter Easy tool has produced the final data files, the script output_fits_sorter.py is responsible for taking the output files and putting them in the 'Output' folder. After that, two simple plotting scripts are available, plot_1d.py and plot_2d.py.

In total, Xshooter Easy runs the following scripts and esorex commands:

input_fits_sorter.py

xsh_mdark_NIR.py
esorex xsh_mdark xsh_mdark.sof
xsh_predict_NIR.py
esorex xsh_predict xsh_predict.sof
xsh_orderpos_NIR.py
esorex xsh_orderpos xsh_orderpos.sof
xsh_mflat_NIR.py
esorex xsh_mflat xsh_mflat.sof
xsh_2dmap_NIR.py
esorex xsh_2dmap xsh_2dmap.sof
xsh_scired_slit_stare_NIR.py
esorex xsh_scired_slit_stare --generate-SDP-format=true xsh_scired_slit_stare.sof

xsh_mbias_VIS.py
esorex xsh_mdark xsh_mdark.sof
xsh_predict_VIS.py
esorex xsh_predict xsh_predict.sof
xsh_orderpos_VIS.py
esorex xsh_orderpos xsh_orderpos.sof
xsh_mflat_VIS.py
esorex xsh_mflat xsh_mflat.sof
xsh_2dmap_VIS.py
esorex xsh_2dmap xsh_2dmap.sof
xsh_scired_slit_stare_VIS.py
esorex xsh_scired_slit_stare --generate-SDP-format=true xsh_scired_slit_stare.sof

xsh_mbias_UVB.py
esorex xsh_mdark xsh_mdark.sof
xsh_predict_UVB.py
esorex xsh_predict xsh_predict.sof
xsh_orderpos_UVB.py
esorex xsh_orderpos xsh_orderpos.sof
xsh_mflat_UVB.py
esorex xsh_mflat xsh_mflat.sof
xsh_2dmap_UVB.py
esorex xsh_2dmap xsh_2dmap.sof
xsh_scired_slit_stare_UVB.py
esorex xsh_scired_slit_stare --generate-SDP-format=true xsh_scired_slit_stare.sof

output_fits_sorter.py

Secondly, all of scripts intended to generate the .sof files use conditions (functions that return true or false) which are found in the files
xsh_conditions_NIR.py
xsh_conditions_VIS.py
xsh_conditions_UVB.py

All of these scripts can cause problems. However, they *have run without error* at least once in testing.


## USEFUL LINKS:

The XSHOOTER pipeline user manual I used to develop this script is found at:
https://ftp.eso.org/pub/dfs/pipelines/instruments/xshooter/xshoo-pipeline-manual-3.8.3.pdf
https://www.eso.org/sci/facilities/paranal/instruments/xshooter/doc/xshooter_Vernet2011.pdf
The XSHOOTER instrument manual is found at:
https://www.eso.org/sci/facilities/paranal/instruments/xshooter/doc/VLT-MAN-ESO-14650-4942_P108v1.pdf

And finally, the official reference paper detailing the XSHOOTER instrument is at:
https://www.eso.org/sci/facilities/paranal/instruments/xshooter/doc/xshooter_Vernet2011.pdf


## CONTACT:

You can send me an email at
gi3100br-s@student.lu.se
or, in case I have finished my education at that university,
bravogianmichele1@gmail.com
