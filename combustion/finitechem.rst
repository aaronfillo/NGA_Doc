##############
finitechem.f90
##############

This module contains all subroutines necessary for the use of the finite-rate chemistry model, including the calculation of species and fluid properties, based on the provided chemical mechanism.

**Module** :f:mod:`finitechem`

.. f:module:: finitechem
    :synopsis: Finite-rate chemistry module.
    
.. f:variable:: R_cst
	
	:type: real
	:attrs: parameter = 8.314462
	
	Ideal gas constant, J/(mol.K).
	
.. f:variable:: small_Y

	:type: real
	:attrs: parameter = 1e-60
	
	Small value for the mass fraction.
	
.. f:variable:: BDCOEFF
	
	:type: real
	:attrs: parameter = 3.0_WP/16.0_WP*SQRT(2.0_WP/Pi*R_cst**3)
	
	Coefficient for the computation of bindary diffusion coefficients.
	
.. f:variable:: CDCOEFF
    
    :type: real
    :attrs: parameter = 0.5_WP*SQRT(5.0_WP/16.0_WP*SQRT(R_cst/Pi))
    
    Coefficient for the computation of reduced collision diameters.
    
.. f:variable:: nReac

	:type: integer
	
	Number of reactions in the chemical mechanism.
	
.. f:variable:: nSpec

	:type: integer
	
	Number of species in the chemical mechanism.
	
.. f:variable:: W_sp
	
	:type: real
	:attrs: pointer
	
	Array of species molecular weight, kg/mol.
	
.. f:variable:: Cp_sp

	:type: real
	:attrs: pointer
	
	Array of species heat capacities, J/mol.K.
	
.. f:variable:: h_sp

	:type: real
	:attrs: pointer
	
	Array of species enthalpies, J/mol.K.
	
.. f:variable:: kOverEps

	:type: real
	:attrs: pointer
	
	Array of species values of :math:`k/\epsilon`.
	
.. f:variable:: muCoeff

	:type: real
	:attrs: pointer
	
	Array of species values of muCoeff, used to calculate viscosity.
	
.. f:variable:: Lewis

	:type: real
	:attrs: pointer
	
	Array of species Lewis numbers.
	
.. f:variable:: Tmin
	
	:type: real
	:attrs: parameter = 295
	
	Clipping value for temperature, often used with max() to determine if a local value of temperature is less than Tmin.
	
.. f:variable:: Tmax

	:type: real
	:attrs: parameter = 3000
	
	Clipping value for temperature, often used with min() to determine if a local value of temperature is greater than Tmax.
	
.. f:variable:: mixavg

	:type: logical
	
	Logical variable, true if the transport model is either Mix Avg or Mix Avg Soret.
	
.. f:variable:: soret

	:type: logical
	
	Logical variable, true if the transport model is Mix Avg Soret.
	
.. f:function:: finitechem_omegamu(temp,omegamu)

	Computes the :math:`\Omega^{(2,2)}` collision integral (cf. FlameMaster/Tspecies.cpp).  Expects a reduced temperature, defined as :math:`Tk/\epsilon`.  The coefficients for the fit are not from a published source.

	:param real temp [in]: Reduced temperature, :math:`Tk/\epsilon`.
	:param real omegamu [out]: :math:`\Omega^{(2,2)}` collision integral.
	
	:from: :f:subr:`finitechem_viscosity` :f:subr:`finitechem_diffusivity` :f:subr:`finitechem_source_zmix`
	
.. f:function:: finitechem_omegaD(temp,omegaD)

	Computes the :math:`\Omega^{(1,1)}` collision integral (cf. FlameMaster/Tspecies.cpp).  Expects a reduced temperature, defined as :math:`Tk/\epsilon`.  The coefficients for the fit are not from a published source.

	:param real temp [in]: Reduced temperature, :math:`Tk/\epsilon`.
	:param real omegaD [out]: :math:`\Omega^{(1,1)}` collision integral.
	
	:from: :f:subr:`finitechem_diffusivity` :f:subr:`finitechem_source_zmix`
	
.. f:function:: finitechem_omegaC(temp,omegaC)

	Computes the ratio of :math:`\Omega^{(1,2)}` and :math:`\Omega^{(1,1)}` collision integrals (cf. FlameMaster/Tspecies.cpp).  This variable is used in the computation of mixture-averaged thermal diffusion coeficients.  Expects a reduced temperature, defined as :math:`Tk/\epsilon`.  The coefficients for the fit are not from a published source.

	:param real temp [in]: Reduced temperature, :math:`Tk/\epsilon`.
	:param real omegaC [out]: :math:`\Omega^{(1,2)}/\Omega^{(1,1)}` ratio.
	
	:from: :f:subr:`finitechem_diffusivity`
    
.. f:subroutine:: finitechem_init
	
	Initializes the finite-rate chemistry module.  This involves storing the number of species, their names, as well as coefficients necessary to compute thermodynamic and transport properties.  Allocates many variables and arrays related to finitechem, such as the species properties (W_sp, kOverEps, muCoeff, Lewis), temporary arrays (Cp_sp, h_sp, tmpS, tmpM, sol, dsol, dsol_n, bindiff) and arrays for the reactions (tmpR1, tmpR2), 
	
	:param names: Array of string that stores the species names.
	:param transport: String that stores the chosen transport model.
	
	:to: :f:subr:`GETNSPECIES` :f:subr:`GETSPECIESNAMES` :f:subr:`GETMOLARMASS` :f:subr:`GETMUCOEFF` :f:subr:`GETKOVEREPS` :f:subr:`parser_read` :f:subr:`GETNREACTIONS`
	
.. f:subroutine:: finitechem_density

	Computes the density of the mixture in the entire domain (including ghost cells) using the equation of state.
	
	:param temp: Temperature, K.
	:param Wmix: Mixture molecular weight, kg/mol.

.. f:subroutine:: finitechem_molefraction(isc,X_)

	Computes the species mole fraction using the mixture molecular weight, species molecular weight, and species mass fraction, everywhere in the domain (including ghost cells).
	
	:param X_: Array of species mole fractions.
	:param W_mix: Mixture molecular weight, kg/mol.
	:param isc: Index for the desired species.

.. f:subroutine:: finitechem_heatcapacity(Cp_mix)

	Computes the mixture heat capacity using the temperature and species heat capacity.
	
	:param Cp_mix: Array of mixture heat capacity, J/mol.K.
	:param temp: Temperature, K.
	:param Cp_sp: Species heat capacity, J/mol.K.
	
.. f:subroutine:: finitechem_viscosity

	Computes the mixture viscosity after compting the species viscosity.  First, the species viscosities are computed using :f:var:`muCoeff` and :f:subr:`finitechem_omegamu`.  Then, the mixture-averaged viscosity is computed using a Wilke's-like formula (cf. App. B in Lapointe et al., Combust. Flame 162 (2015) 3341-3355.)
	
	:param temp: Temperature, K.
	:param W_mix: Mixture molecular weight, kg/mol.
	:param omegamu: Value of the :math:`\Omega^{(2,2)}` collision integral.
	:param tmpS: Temporary array for species viscosities.
	:param VISC: Array of the mixture viscosity, :math:`\mu`, kg/(m.s).
	
	:to: :f:subr:`finitechem_omegamu`	

.. f:subroutine:: finitechem_diffusivity

	Computes the diffusivity coefficients, including thermal diffusion coefficients if the Mix Avg Soret transport model is selected.  First, the species conductivities, :math:`\lambda`, is computed from the Modified Eucken formula.  Then, the mixture-averaged conductivity is computed (cf. Eq. 12.119 in Kee, Coultrin, and Glarborg, Chemically Reacting Flows).  The diffusion coefficients are then pre-set to :math:`\lambda/Cp_mix`, which assumes constant, unity Lewis numbers.  Then, depending on the transport model, the diffusion coefficients are modified.  
	
	If Lewis is chosen (or, equivalently, :f:var:`mixavg` is false), then the diffusion coefficients are divided by the respective species Lewis number.  If Mix Avg or Mix Avg Soret is chosen for the transport model, then binary diffusion coefficients are computed for each species pair and a mixture-averaged diffusion coefficient is computed (cf. Eq. 5-45 in the Chemkin Theory Manual).  Finally, if Mix Avg Soret is the transport model, the thermal diffusion coefficients are computed using a method similar to Chapman and Cowling, 1970 (cf. TEEMO: Schlup paper on DIFFTHERM).
	
	:param isc: Index for species.
	:param jsc: Index for species.
	:param temp: Temperature, K.
    :param W_mix: Mixture molecular weight, kg/mol.
    :param Cp_mix: Mixture specific heat, J/mol.K.
    :param omegamu: :math:`\Omega^{(2,2)}` collision integral.
    :param omegaD: :math:`\Omega^{(1,1)}` collision integral.
    :param Cstar: Ratio of :math:`\Omega^{(1,2)}/\Omega^{(1,1)}` collision integrals.
    :param beta: Coefficient in Modified Eucken formulation for 1.2*omegamu/omegaD.
    :param lambda: Mixture conductivity.
    :param mass: Reduced molecular mass for a pair of species.
    :param diam: Reduced collision diameter for a pair of species.
    :param corr: Correction for the thermal diffusion coefficients so that the sum of all thermal diffusion coefficients is zero at each cell.
    

    :to: :f:subr:`COMPTHERMODATA` :f:subr:`finitechem_omegamu` :f:subr:`finitechem_omegaD` :f:subr:`boundary_update_border`

.. f:subroutine:: finitechem_source_pressure

	Computes the source terms for the temperature equation due to pressure.
	
	:param src: Array of source terms for each scalar.
	:param Cp_mix: Mixture heat capacity, J/mol.K.
	
	:to: :f:subr:`COMPTHERMODATA`

.. f:subroutine:: finitechem_source_diffusion

	Computes the source terms for the species and temperature equations due to diffusion effects.  First, the mixture heat capacity and molecular weight are computed.  Then, the source terms for the temperature and species equations are updated to account for diffusion effects.  For the temperature equation, this is simply :math:`\rho D/Cp_{mix} \nabla T \nabla Cp_{mix}`.  For the species equations, this is the divergence of the gradient terms in the species equations.  Additionally, the temperature equation gets an additional source term due to the enthalpy flux. 
	
	:param Cp_mix: Mixture heat capacity, J/mol.K.
	:param W_mix: Mixture molecular weight, kg/mol.
	:param isc: Index for species.
	:param st: Integer used for a stencil to satisfy the sum of mass fractions to be 1.
	:param flux: Temporary variable to help compute the fluxes for the temperature equation.
	:param sum1: Temporary variable to help compute the fluxes for the temperature equation.
	:param sum2: Temporary variable to help compute the fluxes for the temperature equation.
	
	
	
	:to: :f:subr:`COMPTHERMODATA`

.. f:subroutine:: finitechem_source_soret

.. f:subroutine:: finitechem_source_tubular

.. f:subroutine:: finitechem_source_conduction

.. f:subroutine:: finitechem_source_chemistry

.. f:subroutine:: finitechem_source_implicit

.. f:subroutine:: finitechem_source_zmix

.. f:subroutine:: finitechem_ignite

.. f:subroutine:: finitechem_AT

.. f:subroutine:: finitechem_monitor

.. f:subroutine:: finitechem_check_bounds

.. f:subroutine:: interpol

