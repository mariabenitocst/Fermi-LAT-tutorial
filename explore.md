Obtaining and exploring LAT data for your favorite source
=====================================================

# Software tools

I assume that you all already installed the VM and Virtual Box
overview of software that will be used: Science Tools and enrico
they are installed in the VM and ready to use



Nice explanation of the analysis steps in the FermiPy tutorial


# Extracting the Data

An events file containing the recorded events that correspond to your source of interest, as well as the area surrounding that source. The size of this surrounding "region-of-interest", or ROI, that you choose will depend on the density and brightness of nearby sources, as well as the type of analysis you are performing. There are two types of events files:
Photon Data - contains all information necessary for science analysis with SOURCE, CLEAN, ULTRACLEAN, or ULTRACLEANVETO event classes
Extended Data - contains all event data, including the standard Transient classes (TRANSIENT010 and TRANSIENT020), plus additional quantities produced by the Level 1 analysis
Only one type of event data file is needed to perform LAT science analysis. We recommend use of the photon data file for most purposes (except for the GRB analysis, for which the extended data are required). Event data for large datasets will be divided into multiple files. The next tutorial, Data Preparation, will discuss how to combine those files.
A spacecraft file containing spacecraft position and orientation information at 30 second intervals. This file is required for LAT science analysis. The LAT data server will only generate a single spacecraft file, regardless of the size of the dataset.
Some analyses also require models of the isotropic and Galactic diffuse background models. You can download them from the Background Models page.

This thread leads you through extracting your LAT data files from the FERMI Science Support Center (FSSC) databases. In order to analyze LAT data, you will need:

1. An **events file** containing the recorded events that correspond to your source of interest, as well as the area surrounding that source. The size of this surrounding "region-of-interest", or ROI, that you choose will depend on the density and brightness of nearby sources, as well as the type of analysis you are performing. 

1. A **spacecraft file** containing spacecraft position and orientation information at 30 second intervals. This file is required for LAT science analysis. The LAT data server will only generate a single spacecraft file, regardless of the size of the dataset.

2. Some analyses also require **models of the isotropic and Galactic diffuse background**. You can download them from the Background Models page.

## Procedure 

To select all photons within a circular region around the source:

1. Go to the FSSC's web site data server: [https://fermi.gsfc.nasa.gov/cgi-bin/ssc/LAT/LATDataQuery.cgi](https://fermi.gsfc.nasa.gov/cgi-bin/ssc/LAT/LATDataQuery.cgi)

2. In this thread we will use this set of parameters:

* Object name or coordinates: 193.98, -5.82

* Coordinate system: "J2000"

* Search radius (degrees): 12

* Observation dates: START, 255398400

* Time system: "MET"

* Energy range (MeV): 100, 300000

* LAT data type: "Photon"

* Spacecraft data: "checked"

1. Click on the 'Start Search' button. The 'Query Submitted' webpage will be displayed and provide an estimate of the time to complete the query, as well as a link to the results webpage.

When you go to this new webpage —'LAT Data Query Results' — you may be told that the query is not yet complete. This webpage will show you a link to where the results of your query can be found. When the query is complete, the data file list will include links to the files themselves.

1. Now we have the spacecraft (pointing and livetime history) file and events data file. There may be multiple events files (_PH##), but there should be only a single spacecraft (_SC##) file. We’ll have to save them in order to use the Fermi Science Tools. Since they are not installed in the computers we are using, we will access another computer remotely.

2. Open the terminal in your computer, by pressing crtl + alt + t. Then, type:

<table>
  <tr>
    <td>>> ssh infieri@10.180.1.194 -X
>> infieri@10.180.1.194's password: fermiLAT</td>
  </tr>
</table>


	Now you are accessing a computer with all the tools necessary in this tutorial.

3. Each one will create a directory in this machine. You can use your own name as the name of your directory:

<table>
  <tr>
    <td>>> mkdir YOURNAME
>> cd YOURNAME</td>
  </tr>
</table>


4. Now you can download via wget the files generated on step 4 above. You just need to use the commands available in the bottom of the webpage.

If you want to know the details about the files you just downloaded, you can check:

[https://fermi.gsfc.nasa.gov/ssc/data/analysis/documentation/Cicerone/Cicerone_Data/LAT_Data_Columns.html](https://fermi.gsfc.nasa.gov/ssc/data/analysis/documentation/Cicerone/Cicerone_Data/LAT_Data_Columns.html)

# Data Preparation

Data preparation consists of two steps:

* (*gtselect*): Used to make cuts based on columns in the event data file such as time, energy, position, zenith angle, instrument coordinates, event class, and event type.

* (*gtmktime*): In addition to cutting the selected events, *gtmktime* makes cuts based on the spacecraft file and updates the good time intervals (GTI) extension.

Event selection with *gtselect*

In this section, we look at making basic data cuts using gtselect. By default, *gtselect* prompts for cuts on:

* Time

* Energy

* Position (RA,Dec,radius)

* Maximum Zenith Angle

However, by using hidden parameters defined on the command line, you can also make cuts on:

* Event class ID

* Event type ID (selects on conversion type, angular or energy reconstruction quality)

* Minimum pulse phase

* Maximum pulse phase

If more than one events file was generated by the LAT Data Server, we will need to provide an input file list, in order to use all the event data files in the same analysis. This text file can be generated by typing:

<table>
  <tr>
    <td>>> ls *_PH* > events.txt</td>
  </tr>
</table>


For a simple point source analysis, it is recommended that you only include events with a high probability of being photons. This cut is performed by selecting "source" class events with the the *gtselect* tool by including the hidden parameter *evclass* on the command line. For LAT Pass 8 data, "source" events are specified as event class 128 (the default value).

Additionally, in Pass 8 you can supply the hidden parameter *evtype* (event type) which is a sub-selection on *evclass*. For a simple analysis, we wish to include all front+back converting events within all PSF and Energy subclasses. This is specified as evtype 3 (the default value).

The recommended values for both *evclass* and evtype may change as LAT data processing develops.

Now run *gtselect* to select the data you wish to analyze. For this example, we consider the source class photons within a 12 degree acceptance cone of the blazar 3C 279.

Now, we will divide the class in 3 groups. One will work with every photon received from the source within the energy range chosen before (from 100 to 300000 MeV). Another will work with "low energy" gamma-rays (from 100 to 10000 MeV). The third will work with the “high energy” gamma-rays (from 10000 to 300000 MeV).

We apply the *gtselect* tool to the data file as follows:

<table>
  <tr>
    <td>>> fermi
>> gtselect evclass=128 evtype=3
Input FT1 file[] @events.txt
Output FT1 file[] 3C279_region_filtered.fits
RA for new search center (degrees) (0:360) [0] 193.98
Dec for new search center (degrees) (-90:90) [0] -5.82
radius of new search region (degrees) (0:180) [180] 12
start time (MET in s) (0:) [0] INDEF
end time (MET in s) (0:) [0] INDEF
lower energy limit (MeV) (0:) [30] 100
upper energy limit (MeV) (0:) [300000] 300000
maximum zenith angle value (degrees) (0:180) [180] 90
Done.
prompt></td>
  </tr>
</table>


In this example, we are using the whole energy range (from 100 to 300000 MeV). Use the values according to the energy range you are studying.

If you don't want to make a selection on a given parameter, just enter a zero (0) as the value.

The output from *gtselect* will be a single file (3C279_region_filtered.fits) containing all events from the combined file list (*events.txt*, in this tutorial) that satisfy the other specified cuts.

In this step we also selected the maximum zenith angle value as suggested by the FSSC. Photons coming from the Earth limb are a strong source of background. You can minimize this effect with a zenith angle cut. The value of 90 degrees is suggested for reconstructing events above 100 MeV and provides a sufficient buffer between your ROI and the Earth's limb. In the next step, *gtmktime* will remove any time period that our ROI overlaps this buffer region. While increasing the buffer (reducing zmax) may decrease the background rate from albedo gammas, it will also reduce the amount of time your ROI is completely free of the buffer zone and thus reduce the livetime on the source of interest.

Time Selection with *gtmktime*

You may have noticed that all of these files have a GTI extension in them. Before we look at making selections with the *gtmktime* tool, we should probably clarify what at **Good Time Interval** (GTI) is:

Simply stated, a GTI is a time range when the data can be considered valid. The GTI extension contains a list of these GTI's for the file. Thus the sum of the entries in the GTI extension of a file corresponds to the time when the data in the file is "good".

How are these interpreted for Fermi?

The initial list of GTI's are the times that the LAT was collecting data over the time range you selected. The LAT does not collect data while the observatory is transiting the Southern Atlantic Anomaly (SAA), or during rare events such as software updates or spacecraft maneuvers.

Notes:

* Your object will most likely not be in the field of view during the entire time that the LAT was taking data.

* Additional data cuts made with *gtmktime* will update the GTI's based on the cuts specified in both *gtmktime* and *gtselect*.

*gtmktime* is used to update the GTI extension and make cuts based on spacecraft parameters contained in the spacecraft (pointing and livetime history) file. It reads the spacecraft file and, based on the filter expression and specified cuts, creates a set of GTIs. These are then combined (logical and) with the existing GTIs in the Event data file, and all events outside this new set of GTIs are removed from the file. New GTIs are then written to the GTI extension of the new file.

*gtmktime* also provides the ability to exclude periods when some event has negatively affected the quality of the LAT data. To do this, we select GTIs by using a logical filter for any of the quantities in the spacecraft file. Some possible quantities for filtering data are:

* DATA_QUAL - quality flag set by the LAT instrument team (1 = ok, 2 = waiting review, 3 = good with bad parts, 0 = bad)

* LAT_CONFIG - instrument configuration (0 = not recommended for analysis, 1 = science configuration)

The current *gtmktime* filter expression recommended by the LAT team is: (DATA_QUAL>0)&&(LAT_CONFIG==1).

It is useful to rename the spacecraft file to something easier. Here, we have renamed it to spacecraft.fits (you will need to know the original name of the spacecraft file, try using *ls* to see you files names):

<table>
  <tr>
    <td>>> mv OriginalName_SC00.fits spacecraft.fits</td>
  </tr>
</table>


Here is an example of running *gtmktime* on the 3C 279 filtered events file. 

<table>
  <tr>
    <td>>> gtmktime
Spacecraft data file[] spacecraft.fits
Filter expression[] (DATA_QUAL>0)&&(LAT_CONFIG==1)
Apply ROI-based zenith angle cut[] no
Event data file[] 3C279_region_filtered.fits
Output event file name[] 3C279_region_filtered_gti.fits</td>
  </tr>
</table>


It is also especially important to apply a zenith cut for small ROIs (< 20 degrees), as this brings your source of interest close to the Earth's limb.There are two different methods for handling the complex cut on zenith angle. One method involves excluding time intervals where the buffer zone defined by the zenith cut intersects the ROI from the list of GTIs. In order to do that, you should answer "yes" at the prompt "Apply ROI-based zenith angle cut[]". You can, instead, apply the zenith cut to the livetime calculation latter in the analyses if necessary (we will not get to this point in this tutorial). This is the method that is currently recommended by the LAT team, and is the method we will use most commonly in these analysis threads. To do this, answer "no" at the *gtmktime* prompt “Apply ROI-based zenith angle cut[]”.

The data with all the cuts described above is provided in the file 3C279_region_filtered_gti.fits.
