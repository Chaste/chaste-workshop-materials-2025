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

Second, read the [tutorial on solving bidomain problems](https://chaste.github.io/docs/user-tutorials/runningbidomainsimulations/), **copy this code to a new C++ .hpp test file in your own user project (N.B. your user project will need to be [set up to use the 'heart' component](https://chaste.github.io/docs/user-guides/user-projects/#user-project-guide))**, as we'll use it as a basis for the exercises below, and run this tutorial.

We'll use [Paraview](https://www.paraview.org/) for visualization (there is also the option to output to the [Meshalyzer](https://git.opencarp.org/openCARP/meshalyzer) or [Cmgui](https://github.com/cmiss/cmgui) formats), but since the cell-based workshop has been using Paraview we'll focus on that.

**Note** you need to run a little script on the Chaste VTK output to add time annotations to avoid the need to duplicate/alter the mesh at each time point. See [Using Paraview for Visualizing Cardiac Simulation Ouptut](https://chaste.github.io/docs/user-guides/visualisation-guides/paraview-for-cardiac/).

## Suggested exercises

* Run the tutorials and visualise the results [using Paraview](https://chaste.github.io/docs/user-guides/visualisation-guides/paraview-for-cardiac/).
  * This command should work if you are using the codespace setup: `cd ~/src && python python/utils/AddVtuTimeAnnotations.py ~/output/BidomainTutorial/vtk_output/results.vtu ~/output/BidomainTutorial/vtk_output/annotated.vtu`
  * Then download to your machine and open `annotated.vtu` with Paraview.
* Copy the test in the tutorial and convert it to the analogous monodomain problem - you only need to change 'Bi/bi' to 'Mono/mono' and how you deal with the results Vec
  (which now has the form `V_0 ... V_n`, not `V_0 phi_0_ ... V_n phi_n`). 
**Hint** we suggest you just use the `ReplicatableVector` to deal with the solution (fine for these small problems), not `DistributedVector`). 
Compare with the bidomain simulation visually.
* By calling 
```cpp
monodomain_problem.SetWriteInfo();
```
 before 
```cpp
monodomain_problem.Solve();
```
determine, approximately, by trial and error, and for this monodomain test, the threshold below which the stimulus magnitude is too small to create an action potential. 

* Convert the first 2D bidomain test to a 1D problem, stimulated at one end (a larger magnitude of the stimulus will be required).
* Convert this into a heterogeneous problem where a different cell model is used in a region in centre of the mesh
  * or, if you prefer, adjust a parameter of your choice within the cell model in just one region of the mesh)
* you will need to visualise the results, with and without the change, side-by-side in Paraview to see it. 

**Note:** in the cell factory, you can do something like

```cpp
if ( (x>0.4) && (x<0.6) )
{
    LuoRudyIModel1991OdeSystem* p_luo_rudy_system = new LuoRudyIModel1991OdeSystem(mpSolver, mpZeroStimulus);
    p_luo_rudy_system->SetParameter("some_parameter_name",0.1); // where the parameter names can be seen by looking in the auto-generated model .hpp files.
    return new p_luo_system;
}
```

For more flexibility in tagging new parameters and adjusting what gets generated see the 
[Code Generation From CellML Guide](https://chaste.github.io/docs/user-guides/code-generation-from-cellml/).
