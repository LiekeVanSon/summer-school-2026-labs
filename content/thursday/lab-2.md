---
title: "Lab 2: Stellar Swinging"
author: Lucas de Sá (lead), Annachiara Picco, Mathias Fabry, Lieke van Son
weight: 3
math: true
toc: true
---


<span style="color: #e7876c;">Timing: approximately 1.5 hour divided into 2 x 45 min blocks </span>


<!-- # Lab 2: Stellar Swinging -->
## Overview

<!-- *LUCAS: note on the figures, I'm still missing a normal HR track X CHE HR track figure in Move 1; then I will copy the whole evolutionary diagram to the conclusions to match the Lab 1 style instead of leaving fragments sprinkled throughout*
Lieke: I think the fragments actually work since the landing page of Thursday shows the overview already as well :) -->


In the previous lab, we have seen how stellar expansion leads to mass transfer, which affects their mass, radius, and orbital separation. 

However, *rapid rotation* can fundamentally change this picture. 
Rotation can have dramatic effects on the mass-radius relation, and as a consequence on the occurrence of mass transfer. 
In Lab 2, we will look at the effect of rapid rotation on the structure of $40-100\,\mathrm{M}_\odot$ stars and their BH products. Along the way, we will also learn how to add a brand new history column to our output.


## Move 1: introduction to chemically homogeneous evolution

<!-- *LUCAS: I'm worried this intro is still too long; I cut parts and put some in detail boxes for the curious, but even then I can do a second cutting pass with a bit more time; should I consider keeping the details boxes a risk? It would be better for no one to get too distracted...* 
Lieke: I've cut and simplified text where I could, but please let me know if you disagree -->


![TidalLocking](lab2/figures/intro0tidallylocked.gif)

Rapidly rotating stars become rotationally deformed (oblate), which prevents them from being simultaneously in hydrostatic and thermal equilibrium (the von Zeipel paradox). 
To resolve this, large-scale currents develop throughout the radiative envelope: material sinks toward the center at the equator and rises to the surface at the poles. These **Eddington-Sweet (ES) circulations** circulation[^eddington1929][^sweet1950], or **meridional circulations**,  mix hydrogen from the outer envelope into the convective core, causing the star to evolve as a single well-mixed entity. 
We call this **chemically homogeneous evolution** (CHE).


The time ES timescale (i.e., how long a circulation takes to transport a mass element all the way from core to surface) is roughly

$$\tau_\mathrm{ES} = \frac{\mathrm{\tau_\mathrm{KH}}}{(\Omega/\Omega_\mathrm{c})^2},$$

where $\tau_\mathrm{KH}$ is the star's thermal timescale, $\Omega$ is its angular rotation speed (assuming rigid body rotation), and $\Omega_\mathrm{c}$ is its critical $\Omega$ (at which the centrifugal force equals gravity on the surface). 
For ES circulations to be effective, we need a short ES timescale, and CHE thus requires the star to maintain near-critical rotation throughout the MS.
This can be achieved in a very tight binary where tidal synchronization keeps the star spinning rapidly for the duration of the MS.


![Collapse stages of CHE evolution](lab2/figures/che_diagram1.png)

Because CHE stars lack a core-envelope structure, they never develop the expanding envelope that would otherwise drive mass transfer. Instead, they remain compact during the MS and further contract afterwards.
In the HR diagram, CHE stars evolve to the left (hotter temperatures), rather than expanding to the right as cool RSGs. 
As they evolve, they become progressively hydrogen-free, and you are left with a bare He star and potentially a Wolf-Rayet star.

![Collapse stages of CHE evolution](lab2/figures/che_diagram2.png)

{{<details title="💡Is a He star not a Wolf-Rayet star?" closed="true">}}
`Wolf-Rayet' star is an observational classification; a spectral class defined by broad emission lines in nitrogen, carbon and/or oxygen (and the absence or near-absence of hydrogen lines). 
'Helium star' is a theoretical/structural classification, referring to a star that (for some reason) does not have a hydrogen envelope. 
So while WR stars are typically interpreted as He stars, not all He stars will observationally be classified as WR stars. 
<!-- Those lines are only excited at temperatures significantly higher than those characteristic of MS star surfaces - and therefore we should generally expect our star to spend at least some of its time as a He star with an O-type or O/WN spectrum. 
This last case, a "slash star", is a transitional type between the O-type and the coolest type of WR. -->
{{< /details >}}


<!-- Let's explore the conditions under which a star is able to evolve chemically homogeneously, and calculate the resulting black hole (BH) spins that can be expected from this kind of object.  -->
<!-- Assuming that rotation comes from tidal synchronization, we will vary the initial orbital period of a massive star binary, test what happens if a star is spun-up in the middle of the MS, compute BH spins, and try out variations of AM transport mechanisms. -->

<!-- *LUCAS: looking at this now and at previous labs, I don't think I can fit a theory of 1D implementation of mixing processes here.  I could still put it in slides and explain it live. We should settle this with the labs in hand.* -->
<!-- *Lieke: I din't we will be very much short for time, we can indeed decide when everything else is set in stone, but the lectures should prioritize explaining materials they _need_ to know for the exercises! * -->

## Move 2: starting in *close position*, or CHE stars in the MS
![ClosePosition](lab2/figures/step1close.gif)
 
Since the timescale of ES circulation depends primarily on stellar mass and rotation rate, let's place our massive stars in short-period binaries where tides maintain rapid rotation and see what combinations of mass and orbital period produce CHE. 

> [!Note]
> To get started, set up a work folder for Lab 2, then download to it the `initial_model` folder from **[here](../lab2/initial_model.tar)**  and untar it. Do the same for the `binary_template` folder from **[here](../lab2/binary_template.tar)**.The `initial_model` contains a basic single star setup to generate our ZAMS models. In `binary_template`, the inlists contain most of the settings for our runs, and the `.list` files necessary output. The `src/run_star_extras.f90` contains a custom implementation of stellar winds geared towards CHE stars. We will later go back to the extras file.

With your work folder setup, choose one of the following masses for your stars, which you will carry through to the end of the lab. More massive stars take longer to run, so pick based on how your computer performed in previous labs! 

| $M_1/\mathrm{M}_\odot$ |
| :--------------------- |
| 40 |
| 70 |
| 100 |
| 300 |

You will first generate the ZAMS model for your binary. Go into `initial_model` and find the correct mass setting in `inlist_project`, then compile (`./mk`) and run (`./rn`). Once the run is complete (it should take only a few seconds), copy the produced `zams.mod` to the `binary_template` folder.

{{<details title="Solution" closed="true">}}
Add to `inlist_project`,
```fortran
&controls
    initial_mass = ! your mass
```
{{</details>}}

CHE stars in binaries are amenable to being treated as twins because they are generally expected to have nearly equal masses (when tides are strongest) and suffer near net-zero mass transfer (being compact). 
This allows us to solve only the primary's structure; we'll tell MESA to treat the secondary as if it were identitical to the primary. This cuts our runtime in half!

We will manually adapt the setup in `binary_template` to evolve our stars as twins. MESA already has a setting for this, which you will be able to find in the inlist defaults.

> [!Warning]
> Even when you set the stars to be treated as twins, you must explicitly set the secondary to have the same mass as the primary.

The `inlist_star` file is already set up for a CHE star and to load `zams.mod` as a starting model. In `inlist_project`,
1. Point the primary to `inlist_star`, and set the secondary to have the same starting mass as the primary.
{{<details title="Solution" closed="true">}}
Add 
```fortran
&binary_job
   inlist_names(1) = 'inlist_star' 
```
and
```fortran
&binary_controls
    m2 = ! your chosen mass
```
{{</details>}}
2. Tell MESA to set the stars as twins in the initial model.
{{<details title="Solution" closed="true">}}
Add
```fortran
&binary_job
   change_initial_model_twins_flag = .true.
   new_model_twins_flag = .true. 
```
{{</details>}}



<!-- Regardless of your chosen mass, CHE is always achievable somewhere below $2\,\mathrm{d}$. Within this period range,  -->
3. Pick an initial $P$ from the table below and set it in `inlist_project`. Then `./mk` and `./rn` your binary.

| $P_\mathrm{orb}/\mathrm{d}$ | $P_\mathrm{orb}/\mathrm{d}$ |
| :-------------------------- | :-------------------------- |
| 0.90 | 1.50 |
| 1.00 | 1.60 |
| 1.10 | 1.70 |
| 1.20 | 1.80 |
| 1.30 | 1.90 |
| 1.40 | 2.00 |


{{<details title="Solution" closed="true">}}
Nothing changes from Lab 1 here. Simply include
```fortran
&binary_controls
    initial_period_in_days = ! your choosen period
```
{{</details>}}


As your model is running, answer the following questions:

<!-- Even when your run fails to go CHE , watch the pgplot window and take note of the termination code.  -->
<!-- There is also physics in why stars do *not* go CHE. -->

1. Watch the pgplot window, in particular the HR diagram and the diffusion coefficient plot. Is your star evolving CHE? How can you tell? Look at the different diffusion coefficients, what do they represent, and how do different mixing modes contribute to mixing from center to surface? 
{{<details title="Solution" closed="true">}}
If your star is going CHE, it should move blue-wards in the HR diagram (to the high temperatures, to the left) for almost the entire MS, indicating very little to no expansion. If it starts moving red-wards, MESA will stop the run very soon due to it not going CHE. 

In the diffusion panel, you should be able to find the convective core (dominated by $D_\mathrm{conv}$) and the overshooting layer above it ($D_\mathrm{ovr}$). Everything above it is the radiative envelope, which, if your star is going CHE, is dominated by the ES circulation ($D_\mathrm{ES}$). The large scale picture is: the core is mixed by convection, the envelope by ES circulation, and the two are connected by convective overshooting. The contribution throughout from $D_\mathrm{GSF}$ corresponds to the Goldreich-Schubert-Fricke instability, another rotational instability.

If your star does not go CHE, you might spot a narrow strip above the overshooting region where $D_\mathrm{ES}$ drops to zero before MESA even stops the run, chemically disconnecting core and envelope. Once the difference between the surface and center abundaces of He reachs $0.2$, the run stops.
{{</details>}}


2. How do you think the period range for CHE depends on initial mass? (Recall the definition of $\tau_\mathrm{ES}$.)

{{<details title="Solution" closed="true">}}
The timescale of ES circulation scales with the thermal timescale (shorter for more massive stars) divided by the rotation rate. More massive stars can therefore sustain efficient mixing at slower rotation rates, so $P_{\max}$ increases with mass and the allowed period range widens.
{{</details>}}

3. If your run is stopped before reaching the end of the MS, note the termination message. What was the reason? This should tell you whether you need to move the period up or down to achieve a CHE star.

{{<details title="Solution" closed="true">}}
There are only a few termination messages you should be getting during this run. If you see `Terminate due to primary not evolving homogeneously`, that means your star is not spinning rapidly enough and needs to be in a closer orbit. If you see `termination code: Terminate because of overflowing initial model` (or L2 overflow), it means your stars are so close they would have undergone an episode of likely unstable mass transfer and merged; in this case, you need a wider orbit.
{{</details>}}

<!-- Note that MESA still tracks the Roche lobe geometry even as the mass transfer rate is set to zero. This is what allows us to compute stellar structure only once while still self-consistently tracking the occurence of Roche lobe or L2 overflow.  -->

> [!Warning]
> Regardless of mass, a succesful MS CHE run is not supposed to take more than 8 minutes, potentially no more than 3 min for the low masses. If you picked one of the higher masses and find yourself waiting for longer than this, try a lower mass. 

{{<details title="💡More on very massive stars and winds" closed="true">}}
If you are following the mass-loss rate plot, specially if you picked a greater mass, you might notice it suddenly starts to increase, and the uptick in mass-loss causes your star to dip down in luminositny sharply. 
This behavior is a direct consequence of the inclusion of MS optically-thick winds in our setup, which are normally characteristic of very massive stars ($\gtrsim100\,\mathrm{M}_\odot$), but can get triggered at lower masses for CHE. As a bonus exercise, you may try later to figure out why that is so by looking at the `wyoming_wind` subroutine implemented in the `run_star_extras`.
{{</details>}}

If you ended up picking a period that did _not_ leat to CHE, (i.e., you did not get the `termination code: xa_central_lower_limit`) please pick another period and try again. By the end of this step, you should have an initial mass, orbital period pair that leads to CHE across the MS.
If you run out of time, feel free to check the solution below so you can generate a model for the next step.

{{<details title="Solution: CHE mass, period pairs" closed="true">}}
If you are having trouble, some reliable settings to get CHE are:
| $M_1/\mathrm{M}_\odot$ | $P_\mathrm{orb}/\mathrm{d}$ |
| :--------------------- | :-------------------------- |
| 40 | 1.00 |
| 70 | 1.20 |
| 100 | 1.50 |
| 300 | 1.80 |
{{</details>}}

> [!Bonus]
> If you already found the M,P pair you will carry forward from the tabulated values, and still have time, you can also try to map out exactly the $P_{\min}-P_{\max}$ range for your mass! For some masses you might have to go above the tabulated periods. Remember that for the next steps you should use a pair from the tabulated values only.

---

## Move 3: do the *rock step*, or post-MS
![SpinningUp](lab2/figures/step2rock.gif)

With their high masses and short periods, CHE stars are natural candidates for producing merging binary black hole systems. 
As rapidly rotating stars, CHE stars are natural candidates for producing high-spin BHs, which would stand out from the current, low-spin-dominated, population of merging BBHs. 
To get an estimate of the BH spins produced by CHE stars, we will now take one of our models from the previous sessions, and run it up to He depletion by restarting the run from where we stoppped  with `./re`.


1. First, change the stopping condition from hydrogen to helium depletion. The restarted run will re-read the inlists and pick this up.
{{<details title="Solution" closed="true">}}
In `inlist_star`, change
```fortran
xa_central_lower_limit_species(1) = 'h1'
xa_central_lower_limit(1) = 1d-5
```
to
```fortran
xa_central_lower_limit_species(1) = 'he4'
xa_central_lower_limit(1) = 1d-5
```
{{</details>}}

2. Add the "Dynamo" plot to the middle column so that we can see the evolution of the internal specific angular momentum ($j$) profile.
{{<details title="Solution" closed="true">}}
In the  `&pgstar` section of `inlist_star`, where we had the "abundance-power-mixing" plot we should now have 'Abundance-Power-Mixing-Dynamo', i.e., 
```fortran
Grid2_plot_name(3) = 'Profile_Panels4'
Profile_Panels4_title = 'Abundance-Power-Mixing-Dynamo'
Profile_Panels4_num_panels = 4
Profile_Panels4_yaxis_name(1) = 'Abundance'
Profile_Panels4_yaxis_name(2) = 'Power'
Profile_Panels4_yaxis_name(3) = 'Mixing'
Profile_Panels4_yaxis_name(4) = 'Dynamo'
```
{{</details>}}


Restart your run with `./re`. Watch the new angular momentum plot in the dynamo panel, and the tidal synchronization timescale in the text summary. 

3. Does the angular momentum profile of your star change at all? Comparing with your neighbors, you might find that the answer depends quite a bit on your mass. The initial shape of the profile is characteristic of rigidly-rotating bodies. Does your star retain that shape, or does the profile deform, developing *differential rotation*? --- different layers rotating with different $\Omega$?
{{<details title="Solution" closed="true">}}
Only more massive stars develop differential rotation. This is because the driver of differential rotation is wind mass-loss, which only spins down the surface. More massive stars have higher mass-loss rates, and therefore are able to develop differential rotation.

Post-MS, the tidal synchronization timescale grows by orders of magnitude, while the mass-loss continues to rise. The `p_orb_div_p_rot_1` column in pgplot will tell you whether your star's spin period is shorter or longer than the orbital period.
We see that the synchronization timescale becomes so long that the wind mass-loss becomes the dominant source of torque, and it exclusively spins the star down.
{{</details>}}


4. Now looking at HR diagram and the Kippenhahn diagram: would it be accurate to say that it never expands at all? Take the time to compare your results to neighbors running different masses, both during and after the MS. Do their tracks look very different from yours?

{{<details title="Solution" closed="true">}}

For lower masses there should be a very modest phase of expansion, evident by a short hook towards towards the top-right in the HRD. This is a very brief back-and-forth excursion into larger radii (like a rock step). Higher masses have stronger wind mass loss (you can compare your Mdot panels), and so little envelope is left that there is no expansion.

From the Kippenhahn diagram, we can see that that brief expansion happens after hydrogen shell ignition. During this stage, the envelop responds to further core contraction by expanding, keeping the shell burning rate constant. This is the so-called "mirror effect" of shell burning, which you might have already met in a stellar structure class.

Eventually, the core ignite helium and stops contracting. At this point the envelope contracts again, and the star continues to contract as helium is burned. The "hook" feature makes it very easy to identify H shell ignition and He core ignition, when it is present.
{{</details>}}



The development of differential rotation is closely linked to angular momentum (AM) transport inside the star, which will try to redistribute AM away from high-$j$ to low-$j$ regions. 
So far we have used  the *Tayler-Spruit dynamo*, which for a wide range of masses is enough to keep the star rigidly rotating to the end of He burning.

Note however that wind mass-loss only spins down the surface of the star, but not the core, which will eventually be the seed spin for BH formation. Post-MS, the core is spinning more rapidly than it was at the end of the MS due to the post-MS contraction. To create a slowly spinning BH, it would thus need to efficiently transport the AM outwards.

## Move 4: *open position* into *underarm twirl*, or the He core spin
![OpenPosition](lab2/figures/step3underarm.gif)

We will next use the mass and spin of our CHE stars at the end of He burning to estimate the produced BH masses and spins. 
The BH dimensionless spin parameter, $\chi$ (sometimes called $a$), is defined as 

$$\chi:=\frac{cJ}{GM^2},$$
and takes on values between $0$ (non-spinning) and $1$ (maximally spinning). 


{{<details title="Why $0<\chi<1$?" closed="true">}}

The motivation comes from solving Einstein's equations for the general case of a BH with mass $M$, AM $J$ and electric charge $Q$ (this is called a Kerr-Newman solution). Demanding that the solution include an event horizon around the singularity yields the condition

$$\frac{Q^2}{4\pi\epsilon_0} + \frac{c^2J^2}{GM^2}\leq GM^2.$$

The hypothesis that Nature does not produce so-called *naked singularities* is called the Cosmic Censorship hypothesis and is tacitly assumed everywhere when $\chi<1$ is imposed. This is a hard constraint on the three numbers that fully define a BH (that the three numbers $M$, $J$, $Q$ suffice is called the No Hair Theorem). Further defining,

$$\chi:=\frac{cJ}{GM^2}\leq1,$$

also encodes the assumption that the BH is electrically neutral, which follows reasonably from stellar evolution.
{{</details>}}


We are going to implement a new history column through `run_star_extras` named `chi_he_core` that will track $\chi$ for the He core. 
While our CHE stars are already almost one big He core (such that the core mass and spin are essentially the total mass and spin), explicitly looking for the He core boundary will allow the same column to be used for stars with a hydrogen envelope later.


To implement a new column in `run_star_extras`, we will rely on a few quantities that are internally computed in mesa and available through a `star_info` object, instantiated as `s`. 
Scalar quantities stored in `s` can be recovered as `s% property_name`, while arrays can be recovered as `s% array_name(index or indices)`. The available properties are listed in `$MESA_DIR/star_data/public`. 


> [!Note]
> MESA arrays run from the surface to the center. The index of the innermost "shell" corresponds to the total number of zones into which the star is divided, which is stored as `s% nz`. Fortran arrays can be sliced as `array(index1:index2)`. 

For calculating physical quantities, MESA already includes a large collection of physical constants in CGS in the `const_def` library. This library is already imported by default in `run_star_extras` and its constants, which you can find in `const/public/`, can be used directly. 

<!-- Let's first make sure we understand how to get properties at the He core boundary.  -->
MESA already finds and stores information about the He core boundary for us, so to start we just want to note down how to get the numbers we need to compute $\chi$. 

1. Browse through `star_data/public` to find relevant parameters at the He core boundary. Keep an eye out for any quantities that are not stored in CGS; these cases are highlighted explicitly.
{{<details title="Hint" closed="true">}}
Try running `grep -rin he_core` inside `star_data/public`.
{{</details>}}
{{<details title="Solution" closed="true">}}
We find in `star_data/public/star_data_step_work.inc` the list of available quantities computed by MESA at the He core boundary,
```fortran
! abundance boundaries

real(dp) :: he_core_mass ! baryonic (Msun)
real(dp) :: he_core_radius ! Rsun
real(dp) :: he_core_lgT
real(dp) :: he_core_lgRho
real(dp) :: he_core_L ! Lsun
real(dp) :: he_core_v
real(dp) :: he_core_omega ! (s^-1)
real(dp) :: he_core_omega_div_omega_crit
integer :: he_core_k ! boundary is in this cell
```
While the mass is there, we are missing AM. We have, however, the index at the core boundary, meaning we could compute it if we had the AM profile. 

Note also that `he_core_mass` is in Msun, not grams. We can convert it to the CGS by multiply it by `Msun` later.
{{</details>}}

2. We will need to compute the He core total AM ourselves by integrating the *specific* AM from the center to the He core boundary,
$$J_\mathrm{He} = \int_0^{M_\mathrm{He}}j_\mathrm{rot}\,\mathrm{d}m.$$
Look for the necessary arrays (`j_rot(k)`, and `dm(k)` ) again in `star_data/public`. 
How would you write out a the slice of these arrays that captures the He core? 
{{<details title="Hint" closed="true">}}
Try running `grep -rin "angular momentum"` inside `star_data/public`.
{{</details>}}
{{<details title="Solution" closed="true">}}
The specific AM array is defined in `star_data/public/star_data_step_input.inc`,
```fortran
! rotation
real(dp), pointer, dimension(:) :: j_rot ! (nz)
 ! j_rot(k) is specific angular momentum at outer edge of cell k; = i_rot*omega
```
While the $\mathrm{d}m$ array --- here, the mass per shell --- is in `star_data/public/star_data_work_input.inc`
```fortran
real(dp), pointer :: dm(:)
! dm(k) is baryonic mass of cell k
! dm(k) = s% dq(k)*s% xmstar
```

This means you can call the am slice as:
```fortran
s% j_rot(s% he_core_k : s% nz)
```
and the corresponding mass slice as
```fortran
s% dm(s% he_core_k : s% nz)
```
{{</details>}}

<!-- *LUCAS: I thought it would be nice to let students look for the constants and he core parameters themselves, but since the lab is running long we could also just tell them how to call these properties* -->

3. Tell MESA to expect an extra history column in `src/run_star_extras.f90`

{{<details title="Hint" closed="true">}}
Find the function `how_many_extra_history_columns`
{{</details>}}

{{<details title="Solution" closed="true">}}
Simply increase `how_many_extra_history_columns`. to 2
```fortran
   integer function how_many_extra_history_columns(id)
      integer, intent(in) :: id
      integer :: ierr
      type (star_info), pointer :: s
      ierr = 0
      call star_ptr(id, s, ierr)
      if (ierr /= 0) return
      how_many_extra_history_columns = 2
   end function how_many_extra_history_columns
```
{{</details>}}


4. Add the `chi_he_core` column by changing the `data_for_extra_history_columns` subroutine in `run_star_extras.f90`.  
Remember to look for constants in `const/public` if you need them.
{{<details title="Hint 1: constants" closed="true">}}
You will use `clight` and `standard_cgrav` for constants. Remember that the constants, specific AM and masses are already in CGS! 
{{</details>}}
{{<details title="Hint 2: integration" close="true">}}
Since MESA discretizes stellar structure, your integral will be a sum over the product of the `j_rot` and `dm` arrays. Element-wise array products are implemented in `math_lib` with `dot_product`, and array slices can be taken as `array(i1:i2)`. 
{{</details>}}

{{<details title="Solution" closed="true">}}
In `data_for_extra_history_columns`, compute $J_{\rm He}$ and with that the dimensionless spin parameter of the core. 
Then you can name that column and add the computed spin value. Your final implementation should look approximately like this.
```fortran
   subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
      integer, intent(in) :: id, n
      character (len=maxlen_history_column_name) :: names(n)
      real(dp) :: vals(n)
      integer, intent(out) :: ierr
      real(dp) :: dt, chi_he_core, J_he_core
      type (star_info), pointer :: s
      ierr = 0
      call star_ptr(id, s, ierr)
      if (ierr /= 0) return
      
      ! note: do NOT add the extras names to history_columns.list
      ! the history_columns.list is only for the built-in history column options.
      ! it must not include the new column names you are adding here.
      
      dt = dble(time1 - time0) / clock_rate / 60
      names(1) = 'runtime_minutes'
      vals(1) = dt

      ! NEW
      if (s% he_core_k == 0) then
         ! no He core yet
         chi_he_core = 0d0
      else
         J_he_core = dot_product(s% j_rot(s% he_core_k : s% nz),  s% dm(s% he_core_k : s% nz))
         chi_he_core = clight * J_he_core / (standard_cgrav * s% he_core_mass * s% he_core_mass * Msun * Msun)
      end if

      names(2) = 'chi_he_core'
      vals(2) = chi_he_core

   end subroutine data_for_extra_history_columns
```
{{</details>}}

Once you have included your new column, go ahead and recompile MESA by running `./mk` in your work folder. Remember you have to do this everytime you change any files inside `src/`. If you run into errors while compiling and are not sure what you did wrong, don't hesitate to ask for help!

![Collapse stages of CHE evolution](lab2/figures/che_diagram3.png)

<!-- 
adding our new `chi_he_core` column to the text summary in pgplot, and running a small crowd-sourced exercise at the end of the lab.  -->
Add your newly created column to the text summary in pgplot by modifying the `&pgstar` settings in `inlist_star`.
{{<details title="Solution" closed="true">}}
Luckily the last row on column 4 is free, so it is enough to add,
```fortran
Text_Summary1_name(8,4) = 'chi_he_core'
```
{{</details>}}

For the last MESA run of this lab, we would like to test the effect of different AM transport mechanisms on the final BH spins. 
We can change the AM transport prescription in a few ways. 
Firstly, we can either turn the TS dynamo off completely, or apply a flat intensity modifier through the `am_nu_TS_factor` control in `inlist_star`. 

We can also use an entirely different AM transport model, such as the model from Fuller & Lu (2022)[^fullerlu2022]
While based on the fundamental physical mechanisme (the Tayler instability), their AM transport is significicantly stronger than the standard TS dynamo. 
Note that we have already implemented this model in `run_star_extras`:

```fortran
s% other_am_mixing => TSF_Fuller_Lu22
```

To turn it on, we must modify `inlist_star` so that

```fortran
&controls
am_nu_ST_factor = 0
use_other_am_mixing = .true.
```

Note that pointing the `other_am_mixing` hook is necessary in `run_star_extras` for `use_other_am_mixing` to have an effect. 

Choose one of four AM transport alternatives to implement (or keep). 
**Coordinate with your neighbors pick different ones.** 
The model tag will be used later.
<!-- All variations except No TS should take roughly the same amount of time, for a given mass, but even No TS should take at most an extra minute to run.  -->

| AM transport model | Model tag | Settings |
| :----------------- | :-------- | :------- |
| No TS | nu0 | `am_nu_ST_factor=0`, `use_other_am_mixing=.false.` |
| 0.1x TS | nu01 | `am_nu_ST_factor=0.1d0`, `use_other_am_mixing=.false.` |
| 1x TS | nu1 | `am_nu_ST_factor=1.0d0`, `use_other_am_mixing=.false.` |
| FullerLu | fullerlu | `am_nu_ST_factor=0`, `use_other_am_mixing=.true.` |

1. Run your model!
Because we are changing fundamental physics, this time we will have to restart the entire run from ZAMS, which you can do by simply calling `./rn`. Expect that the run will take at least 10 minutes, which you can use to read ahead on the final crowd-sourcing exercise, or discuss your results with your colleagues.

> [!Warning]
> The entire run should still take 10 to 15 minutes, most of it spend in the MS. If you notice your run is past 10 minutes and your star still has not reached hydrogen depletion (check the center_h1), ask for someone to have a look. For the next step you can always use the final profile from Move 2, which corresponds to the 1x TS model.

Once your run is concluded, you might find that $\chi>1$!? How do you interpret this?

{{<details title="Solution" closed="true">}}
The direct reason why we are able to find $\chi>1$ is that we do not actually have a BH with $\chi>1$. We have a $\chi>1$ He core, which is very far from a relativistic regime. What we are finding is more precisely put as: if all the mass and AM in this He core were to turn into a BH, it would have $\chi>1$. Since we expect this cannot happen in Nature, the conclusion is that *not all* the mass and AM can actually make it into the eventual BH.
{{</details>}}

Regardless of what $\chi$ you find, this is only a very rough approximation for what kind of BH will eventually be produced. We can do one better by accounting for the AM structure of the star.

# Move 4: now switch partners! or crowd-sourcing BH spins
![CrowdDancing](lab2/figures/step4crowd.gif)

The *innermost stable circular orbit* (ISCO) is the smallest stable orbit around a BH, and sets the truncation radius of an accretion disk. Assuming the innermost $2.5\,\mathrm{M}_\odot$ form a seed BH at core-collapse, we can walk outward through the star and ask for each mass shell: is the specific AM $j$ of this shell greater than the corresponding $j_\mathrm{ISCO}$, assuming an orbit around all the mass interior to this point?

![Innermost stable circular orbit, rotating x nonrotating BH](lab2/figures/isco_rxte.jpg)
*Credit: NASA/CXC/M.Weiss*

Anywhere where $j/j_\mathrm{ISCO}>1$ *cannot* be accreted onto the BH without losing AM first and is likely to settle into a disk before being accreted or *ejected*. 
While we cannot simulate a disk, we can estimate the *prompt* BH mass and spin, which is the seed mass + the sum of mass shells fulfilling $j/j_\mathrm{ISCO}>1$.
To do this we start at the layer enclosing $2.5\,\mathrm{M}_\odot$, and find the first layer above it where $j/j_\mathrm{ISCO}=1$. 
Everything between the two can be added to the seed BH. This breaks any sort of trivial mapping between the stellar spin and the BH spin. 

As we will see it is not trivial to form a highly-spinning BH from a highly-spinning star!

A Google Collab notebook has been prepared in order to compute that mass and show you the $j/j_\mathrm{ISCO}$ of all your final models in this [Drive folder](https://drive.google.com/drive/folders/1MrACMSBNhIQRqU4GLg_UQCJomW4FuIE1?usp=sharing).
Please rename your last profile.data file according to the instructions, then upload it to that folder.

> [!Naming]
> In order to create labels, the notebook reads your physical settings from the file name. The file you upload should be named `NAME_mX_AMy.data`, where you should replace `NAME` with your name, `X` with your mass (integer) and `y` with your AM transport model tag from the table above. For example, f I ran the $70\,\mathrm{M}_\odot$ star with the 1x TS AM transport model, I would upload my last profile with the name `lucas_m70_AMnu1.data`.

After uploading your profile, you can try running the notebook online. It should automatically pickup your file (if not, check the naming), add it to $j/j_\mathrm{ISCO}$ plot, and compute the prompt mass and spin.
Can you explain the emerging patterns over mass and AM transport?


<!-- *LUCAS: below is a very wip example of the crowd source plot; I don't have the drive folder setup yet but the plot code works*
![Crowd-sourcing example](lab2/figures/bh_spins_v2_isco.png) -->

## Conclusions

By the end of this lab, we have encountered first-hand the most dramatic difference between CHE and non-CHE stars, and how post-MS evolution can decouple the surface and core rotation rates.  
We have also learned how to compute the BH spin corresponding to a He core, and how that assumption does not trivially hold. 
<!-- As we will return to in Lab 3, accretion can also spin stars up, but under very different circumstances from CHE, and the stellar-BH spin connection remains a fresh topic. -->

## References
[^eddington1929]: [Eddington (1929), Internal Circulation in Rotating Stars, MNRAS, 90, 54–59](https://ui.adsabs.harvard.edu/abs/1929MNRAS..90...54E)
[^sweet1950]: [Sweet (1950), the rigorous follow-up treatment](https://ui.adsabs.harvard.edu/abs/1950MNRAS.110..548S)
[^fullerlu2022]: [Fuller & Lu (2022), Angular momentum transport in massive stars](https://ui.adsabs.harvard.edu/search/q=author%3A%22Fuller%2C%20J%22%20author%3A%22Lu%2C%20W%22%20year%3A2022&sort=date%20desc%2C%20bibcode%20desc&p_=0)