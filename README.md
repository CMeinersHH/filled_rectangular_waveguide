# Filled rectangular waveguide test case - Elmer
## 1.0 Introduction
In this test case, the scattering parameters (reflection and transmission) are calculated for a rectangular waveguide using [Elmer](https://www.csc.fi/web/elmer), an open source multiphysical simulation software. The waveguide is partially filled with dielectric material. There is an analytical solution for this arrangement, which can be calculated using the transfer matrix method, for example.

The test case is very similar to the ["Bent waveguide tutorial" (chapter 18)](https://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerTutorials.pdf). In addition to the implementation of the actual calculation with the ["VectorHelmholtz" (model 20)](https://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf) module, this test case also maps: 
- The meshing of a geometry created in [FreeCad](https://www.freecad.org/) with [Gmsh](https://gmsh.info/)
- The preparation of the mesh in FreeCad for Elmer (UNV file)
- The handling of different materials 
- Post-processing to determine the scattering parameters (i.e. reflection S11 and transmission S21)
- The reflection coefficient obtained on the basis of the field components is compared with the method from the "Bent waveguide tutorial", i.e. with the determination via the energy functional.

## 2.0 Overview of the geometry and results
The following picture shows the setup of the geometry. Thereby, the dielectric waveguide section is highlighted. It is not centered in order to avoid a symmetry with respect to the ports. The cross sections of the waveguide correspond to the "Bent waveguide tutorial". Thus it is not a practical, standard waveguide. However, for the purposes of this test case, it is not relevant. The lengths of the different segments are 150 mm, 200 mm, and 250 mm.

![Geometry of the rectangular waveguide setup](https://github.com/CMeinersHH/filled_rectangular_waveguide/blob/main/images/rw_geometry.png)

In FreeCad, the ["Boolean Fragments"](https://wiki.freecad.org/Part_BooleanFragments) and ["CompoundFilter"](https://wiki.freecad.org/Part_CompoundFilter)-Feature has been used to obtain a compsolid geometry and a resulting mesh which takes into account the inner boundaries between the different materials. The dielectric constant of the inner section is $\varepsilon_r=1.5 \, \text{mm}$. The maximum mesh size is 8 mm.

![Mesh of the arrangement](https://github.com/CMeinersHH/filled_rectangular_waveguide/blob/main/images/mesh.png) 

Thereby, it is possible to group faces and solids in so-called ["mesh groups"](https://wiki.freecad.org/FEM_MeshGroup) such that boundary conditions, materials and equations can be easily applied to the mesh later with Elmer.  

In this example, the transverse-electric propagation mode $H_{10}$ is excited at port 1. 
If a frequency scan of this arrangement is to be performed, the range that is to be selected should start slightly higher than the lower cut-off frequency ($H_{10}$ mode) and slightly lower than the cut-off frequency of the $H_{20}$ mode. This ensures that only single-mode propagation occurs. The lower limit is based on the unfilled waveguide and for the upper frequency the filled waveguide must be considered. Strictly speaking, this results in a range of approximately 1.50..2.44 GHz. A range of 1.6..2.3 GHz has been selected here in order to maintain some distance from these limits. 

The following two figures show the results obtained. I did not work too much on the mesh and the solver settings, thus practically I did not study the influence of mesh settings on the solution. As a general statement, it can be said that the agreement between the simulation and the analytical method is good. Remaining differences can certainly be clarified in greater depth with more effort. The extraction of the scattering parameters is based on the electric field at the port boundaries and, for S11, also on the energy functional for comparison.    

![Plot of S11 in dB](https://github.com/CMeinersHH/filled_rectangular_waveguide/blob/main/images/S11.png) 

![Plot of S21 - amplitude](https://github.com/CMeinersHH/filled_rectangular_waveguide/blob/main/images/S21.png) 

## 3.0 Further discussion on implementation specific aspects
Most of the workflow can be retraced by the "postprocessing.ipynb"- Jupyter notebook file in the repository. For reference, the FreeCad file is also part of the repository. This chapter includes some additional remarks to the code.  

### 3.1 UNV-Export, scaling
The standard unit in FreeCad is millimeters, at least this is my setting. The export of the mesh to a .unv file does not seem to be 100 % consistent with the declaration of the UNV dataset 164. I found a similar discussion here: [Link](https://discourse.salome-platform.org/t/bug-smesh-unv-file-bad-usage-of-units-section-164/591). It follows that Elmer ended up interpreting every millimeter as a meter. To be fair: This is rather a mistake by the export procedure/settings (or maybe the user in front of the computer) than an issue of Elmer. In order to be consistent with the .unv file declarations, I chose to scale all the corresponding entries in the file by a factor of 1/1000. This approach proved to be effective.  

### 3.2 Obtaining scattering parameters
The inport boundary conditions are such that an incoming [wave with amplitude](https://www.elmerfem.org/forum/viewtopic.php?t=4026) $E_{max}=jk_0/k_c$, where $k_0$ is the free space wave number and $k_c$ is the cut-off wave number, is impressed to the waveguide. Thus, the actual field at port 1 is a combination of the incoming and reflected wave. The field at port 2 is the propagating wave that leaves the arrangement. The algorithm, I set up, for determining the reflection and transmission coeefficient relies on the sinusoidal field distribution of the $H_{10}$ mode and will not work properly if other modes are present. I used the ["SaveData" (model 60)](https://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf) to obtain the mean integral values of the complex electric field components on the inport (port 1) and outport (port 2) boundary. These values can then be related to the amplitude of the sinusoidal field distribution that in turn can be related to the incoming wave.   

### 3.3 The analytic solution
This rectangular waveguide only has a length-dependent material change (z-direction). This means that each segment can be described within the framework of general transmission line theory. The basic approach is that there is a forward and backwards travelling wave in each part of the medium. The transitions of the materials are determined by individual reflection and transmission coefficients, which resemble in the transition matrix. The propagation in the medium is modeled by the propagation matrix. For this approach, the wave impedances and propagation constants of the different media need to be calculated. The analytic solution is based on harmonic time dependence of kind $Re\{e^{j\omega t}\}$ whereas the "VectorHelmholtz" module assumes $Re\{e^{-j\omega t}\}$. This should be taken into account when considering the phases. 

### 3.4 Getting the test case to run
The repository already contains all files that are necessary for postprocessing the data. However, if you would like to retrace the whole workflow, do it in the following order: 

1. Open the Freecad File, change to the FEM Workbench, click on the "FEMMeshGmsh"-Object in the model tree, select File->Export and save the mesh as "freecad_mesh_gmsh.unv" in your working directory.

2. Execute the step of scaling the mesh in the "postprocessing.ipynb" notebook. The new mesh is written to "mesh.unv".

3. Convert the mesh to Elmer's format by executing the following line in a terminal: 
    ```bash
    elmergrid 8 2 mesh.unv -autoclean
    ``` 
   The Elmer mesh files will automatically be stored in the mesh subdirectory.

4. Compile the "emparams.f90" file which is needed to set the frequency dependent boundary conditions for the sweep. On a Windows machine, it should be:
    ```bash
    elmerf90 -o emparams.dll emparams.f90
    ```  
    while on a linux machine (but I did not test that):
    ```bash
    elmerf90 -o emparams.so emparams.f90
    ``` 
    If you need more information about the contents of the file "emparams.f90", take at look at [this ressource](https://github.com/ElmerCSC/elmer-elmag/tree/main/BandpassFilter) or even have a look at the basic programming section (section 19) of the [Elmer solver manual](https://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerSolverManual.pdf).

5. Invoke the simulation by executing: 
    ```bash
    elmersolver case.sif
    ```
    The execution of the simulation is not parallelized at all. Expect an execution time of about 1 hour on a reasonable fast CPU. 

## 4.0 Tested on my Windows 11 machine with
- Elmer 9.0
- Freecad 0.21.1 (incl. Gmsh 4.11.0)
- Python 3.11.4
- Jupyter Extension for Visual Studio Code 


