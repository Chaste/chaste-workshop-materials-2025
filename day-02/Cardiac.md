<!--
# Writing a simple test

Create a new test file in `heart/test`. Name your new file `Test[Something].hpp`.

Read the tutorial on [writing tests](https://chaste.github.io/docs/user-tutorials/writingtests/). 
Write a simple test that, for example, calculates $\sum_{n=1}^N \frac{1}{n^2}$, for some large $N$, and uses

```
TS_ASSERT_DELTA(my_answer, right_answer, tolerance);
```
to verify that the answer is $\frac{\pi^2}{6}$, to within some tolerance. 

**Note 1:** Recall that in C or C++, if `n>1` is an `int`/`unsigned` `1/(n*n)` will be zero - to get the expected answer, you must write `1.0/(n*n)`

**Note 2:** pi is `M_PI`
-->

# Solving ODEs and PDEs (optional)

Read the [tutorials on solving ODEs and *linear* PDEs](https://chaste.github.io/docs/user-tutorials/). How much time you spend in these will depend on whether you have already spent time installing Chaste, your familiarity with C++ and in particular abstract classes and inheritence, and what you aim to use Chaste for - it may be sensible to skip these tutorials or only look at them briefly. However, they will give an understanding of the 'core' functionality in Chaste and also introduce the components used in cardiac problems - monodomain and bidomains problems are composed of linear PDEs (with a nonlinear source term) coupled to a set of ODEs.

Run the tutorials, then use them as a basis for the following exercises.

# Solving Cardiac Problems

First, have a look at the [Single Cell Cardiac Tutorial](https://chaste.github.io/docs/user-tutorials/singlecellsimulation/), and check that all runs OK on your own install of Chaste (in the build folder, run `ctest -R TestSingleCellSimulationTutorial`).

Second, read the [tutorial on solving bidomain problems](https://chaste.github.io/docs/user-tutorials/runningbidomainsimulations/). **It is highly recommended that you do not edit ```heart/test/tutorials/TestRunningBidomainSimulationsTutorial.hpp``` directly, but copy this code to a new C++ .hpp test file in your own user project (N.B. your user project will need to be [set up to use the 'heart' component](https://chaste.github.io/docs/user-guides/user-projects/#user-project-guide))**, as we'll use it as a basis for the exercises below, and run this tutorial.

We'll use [Paraview](https://www.paraview.org/) for visualization (there is also the option to output to the [Meshalyzer](https://git.opencarp.org/openCARP/meshalyzer) or [Cmgui](https://github.com/cmiss/cmgui) formats), but since the cell-based workshop has been using Paraview we'll focus on that.



## Suggested exercises

* Run the tutorials and visualise the results [using Paraview](https://chaste.github.io/docs/user-guides/visualisation-guides/paraview-for-cardiac/). N.B. you can comment out the lines to output in the other visualizer formats to save a bit of unnecessary file I/O and results download to your local machine.
  * Note that tests can be run from the `build` folder via `ctest` or just by running the executable, e.g. `./heart/test/TestRunningBidomainSimulationsTutorial`
  * To run executables in parallel (on multiple processors) we use MPI, e.g. `mpirun -np 3 heart/test/TestRunningBidomainSimulationsTutorial` to run on 3 processors (note this might actually be slower than running on one process for very small meshes!)
> [!IMPORTANT]
> you need to run a little `AddVtuTimeAnnotations.py` script on the Chaste VTK output to add time annotations to avoid the need to duplicate the mesh at each time point as you might do for mechanics simulations. 

 * This command should work for VTK time annotations if you are in the `build` folder and using the codespace/docker setup: `python python/utils/AddVtuTimeAnnotations.py ~/output/BidomainTutorial/vtk_output/results.vtu ~/output/BidomainTutorial/vtk_output/annotated.vtu` but note that, on other setups, your output [may be in a different folder.](https://chaste.github.io/docs/user-tutorials/#wheres-my-file-output)
  * Then download to your machine by right clicking the folder in VS Code, and open `annotated.vtu` with Paraview. See [Using Paraview for Visualizing Cardiac Simulation Output](https://chaste.github.io/docs/user-guides/visualisation-guides/paraview-for-cardiac/).
> [!IMPORTANT]
> If you are running your own installation of Chaste, and Paraview shows the error ```Error parsing XML in stream at line...``` then you are likely to have hit [a known library incompatibility bug](https://github.com/Chaste/Chaste/issues/249) in recent Ubuntu distributions. Ask for help in fixing your installation (or continue with a codebase/docker installation). 
* Have a look at the auto-converted-at-compile-time CellML file which is in `build/heart/src/odes/cellml/LuoRudy1991.cpp`. Scroll down to the bottom method which sets the 'tagged' parameter names and values, and work out which parameters have been tagged in the CellML file. The 'tagging' means these parameters are treated differently to normal constants in the equations, and are given names so that they can be altered using public methods (e.g.)
  
```p_cell->SetParameter("membrane_fast_sodium_current_conductance", 0.0);```

* Try altering the 'Cell Factory' at the top of ```TestRunningBidomainSimulationsTutorial.hpp``` to use this method to set the sodium channel conductance to zero in one corner of the mesh (x >= 0.05 and y <= 0.02). Visualize the new spead of the activation wave (because a lot of the charge here moves by diffusion, this region still depolarises, but more slowly!). For more flexibility in tagging new parameters and adjusting what gets generated, see the 
[Code Generation From CellML Guide](https://chaste.github.io/docs/user-guides/code-generation-from-cellml/).

> [!TIP]
> in the cell factory, you can do something like
> ```cpp
> if ( (x>0.4) && (x<0.6) )
> {
>     CellLuoRudy1991FromCellML* p_luo_rudy_system = new CellLuoRudy1991FromCellML(mpSolver, mpZeroStimulus);
>     p_luo_rudy_system->SetParameter("membrane_fast_sodium_current_conductance",0.0);
>     return p_luo_rudy_system;
>}
>```

* Make a second test method within the tutorial ```TestRunningBidomainSimulationsTutorial.hpp``` file and convert it to the analogous monodomain problem - you only need to change 'Bi/bi' to 'Mono/mono'. Note, you'll need to `#include MonodomainProblem.hpp` at the top of the file.

(If you were using the results vector within C++, be aware that indexing will be different for voltage as it now has the form `V_0 ... V_n`, not `V_0 phi_0_ ... V_n phi_n`).

> [!NOTE]
> if you need to do anything to examine/print the solution in the C++ file, we suggest you just use the `ReplicatableVector` to deal with the solution (replicated across all processes, fine for these small problems), not `DistributedVector` just because the output from the latter can be confusing if you aren't used to dealing with parallel computing). 


* Calling 
```cpp
monodomain_problem.SetWriteInfo();
```
 before 
```cpp
monodomain_problem.Solve();
```
makes the program print out `[min(V), max(V)]` at each output time. It can be a simple way to see whether waves are propagating without even having to visualise the output. 
Using this `SetWriteInfo()` call determine, approximately, by trial and error, the threshold below which the stimulus magnitude is too small to create a propagating action potential. 

The default end time is only 5ms here, so we just see the activation wave. If you want to see the repolarisation wave you need to run until time 350ms.

## Next Steps

Have a look at the rest of the [Cardiac Tutorials](https://chaste.github.io/docs/user-tutorials/#cardiac) which provide a good overview of all Chaste's cardiac capabilities...

