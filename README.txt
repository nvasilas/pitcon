!*****************************************************************************80
!
!! PITCON is the user-interface routine for the continuation code.
!
!  A) Introduction:
!
!  PITCON solves nonlinear systems with one degree of freedom.
!
!  PITCON is given an N dimensional starting point X, and N-1 nonlinear
!  functions F, with F(X) = 0.  Generally, there will be a connected
!  curve of points Y emanating from X and satisfying F(Y) = 0.  PITCON
!  produces successive points along this curve.
!
!  The program can be used to study many sorts of parameterized problems,
!  including structures under a varying load, or the equilibrium
!  behavior of a physical system as some quantity is varied.
!
!  PITCON is a revised version of ACM TOMS algorithm 596.
!
!  Both versions are available via NETLIB, the electronic software
!  distribution service.  NETLIB has the original version in its TOMS
!  directory, and the current version in its CONTIN directory.
!  For more information, send the message "send index from contin"
!  to "netlib@research.att.com".
!
!
!  B) Acknowledgements:
!
!  PITCON was written by
!
!    Professor Werner C Rheinboldt and John Burkardt,
!    Department of Mathematics and Statistics
!    University of Pittsburgh,
!    Pittsburgh, Pennsylvania, 15260, USA.
!
!  The original work on this package was partially supported by the National
!  Science Foundation under grants MCS-78-05299 and MCS-83-09926.
!
!
!  C) Overview:
!
!  PITCON computes a sequence of solution points along a one dimensional
!  manifold of a system of nonlinear equations F(X) = 0 involving NVAR-1
!  equations and an NVAR dimensional unknown vector X.
!
!  The operation of PITCON is somewhat analogous to that of an initial value
!  ODE solver.  In particular, the user must begin the computation by
!  specifying an approximate initial solution, and subsequent points returned
!  by PITCON lie on the curve which passes through this initial point and is
!  implicitly defined by F(X) = 0.  The extra degree of freedom in the system is
!  analogous to the role of the independent variable in a differential
!  equations.
!
!  However, PITCON does not try to solve the algebraic problem by turning it
!  into a differential equation system.  Unlike differential equations, the
!  solution curve may bend and switch back in any direction, and there may be
!  many solutions for a fixed value of one of the variables.  Accordingly,
!  PITCON is not required to parametrize the implicitly defined curve with a
!  fixed parameter.  Instead, at each step, PITCON selects a suitable variable
!  as the current parameter and then determines the other variables as
!  functions of it.  This allows PITCON to go around relatively sharp bends.
!  Moreover, if the equations were actually differentiated - that is, replaced
!  by some linearization - this would introduce an inevitable "drift" away from
!  the true solution curve.  Errors at previous steps would be compounded in a
!  way that would make later solution points much less reliable than earlier
!  ones.  Instead, PITCON solves the algebraic equations explicitly and each
!  solution has to pass an acceptance test in an iterative solution process
!  with tolerances provided by the user.
!
!  PITCON is only designed for systems with one degree of freedom.  However,
!  it may be used on systems with more degrees of freedom if the user reduces
!  the available degrees of freedom by the introduction of suitable constraints
!  that are added to the set of nonlinear equations.  In this sense, PITCON may
!  be used to investigate the equilibrium behavior of physical systems with
!  several degrees of freedom.
!
!  Program options include the ability to search for solutions for which a
!  given component has a specified value.  Another option is a search for a
!  limit or turning point with respect to a given component; that is, of a
!  point where this particular solution component has a local extremum.
!
!  Another feature of the program is the use of two work arrays, IWORK and
!  RWORK.  All information required for continuing any interrupted computation
!  is saved in these two arrays.
!
!
!  D) PITCON Calling Sequence:
!
!  subroutine PITCON(DF,FPAR,FX,IERROR,IPAR,IWORK,LIW,NVAR,RWORK,LRW,XR,SLVNAM)
!
!  On the first call, PITCON expects a point XR and a routine FX defining a
!  nonlinear function F.  Together, XR and FX specify a curve of points Y
!  with the property that F(Y) = 0.
!
!  On the first call, PITCON simply verifies that F(XR) = 0.  If this is not
!  the case, the program attempts to correct XR to a new value satisfying
!  the equation.
!
!  On subsequent calls, PITCON assumes that the input vector XR contains the
!  point which had been computed on the previous call.  It also assumes that
!  the work arrays IWORK and RWORK contain the results of the prior
!  calculations.  PITCON estimates an appropriate stepsize, computes the
!  tangent direction to the curve at the given input point, and calculates a
!  predicted new point on the curve.  A form of Newton's method is used to
!  correct this point so that it lies on the curve.  If the iteration is
!  successful, the code returns with a new point XR.  Otherwise, the stepsize
!  may be reduced, and the calculation retried.
!
!  Aside from its ability to produce successive points on the solution curve,
!  PITCON may be asked to search for "target points" or "limit points".
!  Target points are solution vectors for which a certain component has a
!  specified value.  One might ask for all solutions for which XR(17) = 4.0, for
!  instance.  Limit points occur when the curve turns back in a given
!  direction, and have the property that the corresponding component of the
!  tangent vector vanishes there.
!
!  If the user has asked for the computation of target or limit points, then
!  PITCON will usually proceed as described earlier, producing a new
!  continuation point on each return.  But if a target or limit point is
!  found, such a point is returned as the value of XR, temporarily interrupting
!  the usual form of the computation.
!
!
!  E) Overview of PITCON parameters:
!
!  Names of routines:
!
!    DF     Input,        external DF, evaluates the Jacobian of F.
!    FX     Input,        external FX, evaluates the function F.
!    SLVNAM Input,        external SLVNAM, solves the linear systems.
!
!  Information about the solution point:
!
!    NVAR   Input,        integer NVAR, number of variables, dimension of XR.
!    XR     Input/output, real XR(NVAR), the current solution point.
!
!  Workspace:
!
!    LIW    Input,        integer LIW, the dimension of IWORK.
!    IWORK  Input/output, integer IWORK(LIW), work array.
!
!    LRW    Input,        integer LRW, the dimension of RWORK.
!    RWORK  Input/output, real RWORK(LRW), work array.
!
!    FPAR   "Throughput", real FPAR(*), user defined parameter array.
!    IPAR   "Throughput", integer IPAR(*), user defined parameter array.
!
!  Error indicator:
!
!    IERROR Output,       integer IERROR, error return flag.
!
!  Modified:
!
!    16 May 2008
!
!  Author:
!
!    John Burkardt
!
!  Parameters:
!
!  DF     Input, external DF, the name of the Jacobian evaluation routine.
!         This name must be declared external in the calling program.
!         DF is not needed if the finite difference option is used
!         (IWORK(9) = 1 or 2). In that case, only a dummy name is needed for DF.
!         Otherwise, the user must write a routine which evaluates the
!         Jacobian matrix of the function FX at a given point X and stores it
!         in the FJAC array in accordance with the format used by the solver
!         specified in SLVNAM.
!         In the simplest case, when the full matrix solverDENSLV solver
!         provided with the package is used, DF must store  D F(I)/D X(J) into
!         FJAC(I,J).
!         The array to contain the Jacobian will be zeroed out before DF is
!         called, so that only nonzero elements need to be stored.  DF must
!         have the form:
!           subroutine DF(NVAR,FPAR,IPAR,X,FJAC,IERROR)
!           NVAR   Input, integer NVAR, number of variables.
!           FPAR   Input, real FPAR(*), vector for passing parameters.
!                  This vector is not used by the program, and is only provided
!                  for the user's convenience.
!           IPAR   Input, integer IPAR(*), vector for passing integer
!                  parameters.  This vector is not used by the program, and is
!                  only provided for the user's convenience.
!           X      Input, real X(NVAR), the point at which the
!                  Jacobian is desired.
!           FJAC   Output, real FJAC(*), array containing Jacobian.
!                  If DENSLV is the solver:  FJAC must be dimensioned
!                  FJAC(NVAR,NVAR) as shown above, and DF sets
!                  FJAC(I,J) = D F(I)/DX(J).
!                  If BANSLV is the solver:  the main portion of the Jacobian,
!                  rows and columns 1 through NVAR-1, is assumed to be a banded
!                  matrix in the standard LINPACK form with lower bandwidth ML
!                  and upper bandwidth MU.  However, the final column of the
!                  Jacobian is allowed to be full.
!                  BANSLV will pass to DF the beginning of the storage for
!                  FJAC, but it is probably best not to doubly dimension FJAC
!                  inside of DF, since it is a "hybrid" object.  The first
!                  portion of it is a (2*ML+MU+1, NEQN) array, followed by a
!                  single column of length NEQN (the last column of the
!                  Jacobian).  Thus the simplest approach is to declare FJAC to
!                  be a vector, and then then to store values as follows:
!
!                    If J is less than NVAR, then
!                      if I-J .LE. ML and J-I .LE. MU,
!                        set K = (2*ML+MU)*J + I - ML
!                        set FJAC(K) = D F(I)/DX(J).
!                      else
!                        do nothing, index is outside the band
!                    endif
!                    Else if J equals NVAR, then
!                      set K = (2*ML+MU+1)*(NVAR-1)+I,
!                      set FJAC(K) = D F(I)/DX(J).
!                  endif.
!           IERROR Output, integer IERROR, error return flag.  DF should set
!                  this to 0 for normal return, nonzero if trouble.
!
!  FPAR   Input/output, real FPAR(*), a user defined parameter array.
!         FPAR is not used in any way by PITCON.  It is provided for the user's
!         convenience.  It is passed to DF and FX, and hence may be used to
!         transmit information between the user calling program and these user
!         subprograms. The dimension of FPAR and its contents are up to the
!         user.  Internally, the program declares DIMENSION FPAR(*) but never
!         references its contents.
!
!  FX     Input, external FX, the name of the routine to evaluate the function.
!         FX computes the value of the nonlinear function.  This name must be
!         declared external in the calling program.  FX should evaluate the
!         NVAR-1 function components at the input point X, and store the result
!         in the vector FVEC.  An augmenting equation will be stored in entry
!         NVAR of FVEC by the PITCON program.  FX should have the form:
!
!           subroutine FX ( NVAR, FPAR, IPAR, X, FVEC, IERROR )
!
!           Input, integer NVAR, number of variables.
!           Input/output, real FPAR(*), user parameters.
!           Input/output, integer IPAR(*), array of user parameters.
!           Input, real X(NVAR), the point of evaluation.
!           Output, real FVEC(NVAR-1), the value of the function at X.
!           Output, integer IERROR, 0 for no errors, nonzero for an error.
!
!  IERROR Output, integer IERROR, error return flag.
!
!         On return from PITCON, a nonzero value of IERROR is a warning of some
!         problem which may be minor, serious, or fatal.
!
!         0, No errors occurred.
!
!         1, Insufficient storage provided in RWORK and IWORK, or NVAR is less
!            than 2.  This is a fatal error, which occurs on the first call to
!            PITCON.
!
!         2, A user defined error condition occurred in the FX or DF
!          subroutines.  PITCON treats this as a fatal error.
!
!         3, A numerically singular matrix was encountered.  Continuation
!            cannot proceed without some redefinition of the problem.  This is
!            a fatal error.
!
!         4, Unsuccessful corrector iteration.  Loosening the tolerances
!            RWORK(1) and RWORK(2), or decreasing the maximum stepsize RWORK(4)
!            might help.  This is a fatal error.
!
!         5, Too many corrector steps.  The corrector iteration was proceeding
!            properly, but too slowly.  Increase number of Newton steps
!            IWORK(17), increase the error tolerances RWORK(1) or RWORK(2), or
!            decrease RWORK(4).  This is a fatal error.
!
!         6, Null tangent vector.  A serious error which indicates a data
!            problem or singularity in the nonlinear system.  This is a fatal
!            error.
!
!         7, Root finder failed while searching for a limit point.
!            This is a warning.  It means that the limit point
!            computation has failed, but the main computation (computing the
!            continuation curve itself) may continue.
!
!         8, Limit point iteration took too many steps.  This is a warning
!            error.  It means that the limit point computation has failed, but
!            the main computation (computing the continuation curve itself) may
!            continue.
!
!         9, Target point calculation failed.  This generally means that
!            the program detected the existence of a target point, set up
!            an initial estimate of its value, and applied the corrector
!            iteration, but that this corrector iteration failed.
!            This is a only a warning message.  PITCON can proceed to compute
!            new points on the curve.  However, if the target point was
!            really desired, PITCON has no automatic recovery method to
!            retry the calculation.  The best prescription in that case
!            is to try to guarantee that PITCON is taking smaller steps
!            when it detects the target point, which you may do by reducing
!            HMAX, stored as RWORK(4).
!
!         10, Undiagnosed error condition.  This is a fatal error.
!
!  IPAR   Input/output, integer IPAR(*), user defined parameter array.
!
!         IPAR is not used in any way by PITCON.  It is provided for the user's
!         convenience in transmitting parameters between the calling program
!         and the user routines FX and DF.  IPAR is declared in the PITCON
!         program and passed through it to FX and DF, but otherwise ignored.
!         Note, however, that if BANSLV is used for the solver routine, then
!         IPAR(1) must contain the lower bandwidth, and IPAR(2) the upper
!         bandwidth of the Jacobian matrix.
!
!  IWORK  Input/output, integer IWORK(LIW).  Communication and workspace array.
!
!         The specific allocation of IWORK is described in the section devoted
!         to the work arrays.  Some elements of IWORK MUST be set by the user,
!         others may be set to change the way PITCON works.
!
!  LIW    Input, integer LIW, the dimension of IWORK.
!
!         The minimum acceptable value of LIW depends on the solver chosen,
!         but for either DENSLV or BANSLV, setting LIW = 29+NVAR is sufficient.
!
!  NVAR   Input, integer NVAR, the number of variables, the dimension of X.
!
!         This is, of course, one greater than the number of equations or
!         functions.  NVAR must be at least 2.
!
!  RWORK  Input/output, real RWORK(LRW), work array.
!
!         The specific allocation of RWORK is described in the section
!         devoted to the work arrays.
!
!  LRW    Input, integer LRW, the dimension of RWORK.
!
!         The minimum acceptable value depends heavily on the solver options.
!         There is storage required for scalars, vectors, and the Jacobian
!         array.  The minimum acceptable value of LRW is the sum of three
!         corresponding numbers.
!
!         For DENSLV with user-supplied Jacobian,
!
!           LRW = 29 + 4*NVAR + NVAR*NVAR.
!
!         For DENSLV with internally approximated Jacobian,
!
!           LRW = 29 + 6*NVAR + NVAR*NVAR.
!
!         For BANSLV, with a Jacobian matrix with upper bandwidth MU and lower
!         bandwidth ML, and NBAND = 2*ML+MU+1, with user supplied Jacobian,
!
!           LRW = 29 + 6*NVAR + (NVAR-1)*NBAND.
!
!         For BANSLV with internally approximated Jacobian,
!
!           LRW = 29 + 9*NVAR + (NVAR-1)*NBAND.
!
!  XR     Input/output, real XR(NVAR), the current solution point.
!
!         On the first call, the user should set XR to a starting point which
!         at least approximately satisfies F(XR) = 0.  The user need never
!         update XR again.
!
!         Thereafter, on each return from the program with IERROR = 0, XR will
!         hold the most recently computed point, whether a continuation, target
!         or limit point.
!
!  SLVNAM Input, external SLVNAM, the name of the solver to use on linear
!         systems.
!
!         The linear systems have the form A*x = b, where A is the augmented
!         Jacobian matrix.  A will be square, and presumably nonsingular.
!         The routine must return the value of the solution x.
!
!         Two possible choices for SLVNAM are "DENSLV" and "BANSLV", which are
!         the names of routines provided with the package.  DENSLV is
!         appropriate for a full storage jacobian, and BANSLV for a jacobian
!         which is banded except for the last column.
!
!         The advanced user may study the source code for these two routines
!         and write an equivalent solver more suited to a given problem.
!
!
!  G) The Integer Work Array IWORK:
!
!  Input to PITCON includes the setting of some of the entries in IWORK.
!  Some of this input is optional.  The user input section of IWORK involves
!  entries 1 through 9, and, possibly also 17.
!
!  IWORK(1) must be set by the user.  All other entries have default values.
!
!
!  IWORK(1)        On first call only, the user must set IWORK(1) = 0.
!                  Thereafter, the program sets IWORK(1) before return to
!                  explain what kind of point is being returned.  This return
!                  code is:
!
!                      1 return of corrected starting point.
!                      2 return of continuation point.
!                      3 return of target point.
!                      4 return of limit point.
!
!                  NOTE:  At any time, PITCON may be called with a negative
!                  value of IWORK(1). This requests a check of the
!                  jacobian routine against a finite difference approximation,
!                  or a printout of the jacobian or its approximation.
!
!                      -1, compare Jacobian against forward difference,
!                          print maximum discrepancy only.
!                      -2, compare Jacobian against central difference,
!                          print maximum discrepancy only.
!                      -3, compare Jacobian against forward difference,
!                          print out the entire discrepancy matrix.
!                      -4, compare Jacobian against central difference,
!                          print out the entire discrepancy matrix.
!                      -5, compute forward difference Jacobian,
!                          print maximum entry only.
!                      -6, compute central difference Jacobian,
!                          print maximum entry only.
!                      -7, compute forward difference Jacobian,
!                          print entire matrix.
!                      -8, compute central difference Jacobian,
!                          print entire matrix.
!                      -9, request user supplied Jacobian,
!                          print maximum entry only.
!                     -10, request user supplied Jacobian,
!                          print entire matrix.
!
!                  Before a call with negative IWORK(1), the current value of
!                  IWORK(1) should be saved, and then restored to the previous
!                  value after the call, in order to resume calculation.
!
!                  IWORK(1) does not have a default value.  The user MUST set
!                  it.
!
!  IWORK(2)        IPC, the component of the current continuation point XR
!                  which is to be used as the continuation parameter.  On first
!                  call, the program is willing to use the index NVAR as a
!                  default, but the user should set this value if better
!                  information is available.
!
!                  After the first call, the program sets this value for each
!                  step automatically unless the user prevents this by setting
!                  the parameterization option IWORK(3) to a non-zero valus.
!                  Note that a poor choice of IWORK(2) may cause the algorithm
!                  to fail.  IWORK(2) defaults to NVAR on the first step.
!
!  IWORK(3)        Parameterization option.  The program would prefer to be
!                  free to choose a new local parameter from step to step.
!                  The value of IWORK(3) allows or prohibits this action.
!                  IWORK(3) = 0 allows the program to vary the parameter,
!                  IWORK(3) = 1 forces the program to use whatever the contents
!                  of IWORK(2) are, which will not be changed from the user's
!                  input or the default.  The default is IWORK(3) = 0.
!
!  IWORK(4)        Newton iteration Jacobian update option.
!                  0, the Jacobian is reevaluated at every step of the
!                     Newton iteration.  This is costly, but may result in
!                     fewer Newton steps and fewer Newton iteration rejections.
!                  1, the Jacobian is evaluated only on the first and
!                     IWORK(17)-th steps of the Newton process.
!                  2, the Jacobian is evaluated only when absolutely
!                     necessary, namely, at the first step, and when the
!                     process fails. This option is most suitable for problems
!                     with mild nonlinearities.
!
!                  The default is IWORK(4) = 0.
!
!  IWORK(5)        IT, target point index.  If IWORK(5) is not zero, it is
!                  presumed to be the component index between 1 and NVAR for
!                  which target points are sought.  In this case, the value of
!                  RWORK(7) is assumed to be the target value.  The program
!                  will monitor every new continuation point, and if it finds
!                  that a target point may lie between the new point and the
!                  previous point, it will compute this target point and
!                  return.  This target point is defined by the property that
!                  its component with the index prescribed in IWORK(5) will
!                  have the value given in RWORK(7).  For a given problem there
!                  may be zero, one, or many target points requested.
!                  The default of IWORK(5) is 0.
!
!  IWORK(6)        LIM, the limit point index.  If IWORK(6) is nonzero, then
!                  the program will search for limit points with respect to
!                  the component with index IWORK(6); that is, of points for
!                  which the IWORK(6)-th variable has a local extremum, or
!                  equivalently where the IWORK(6)-th component of the tangent
!                  vector is zero.  The default of IWORK(6) is zero.
!
!  IWORK(7)        IWRITE, which controls the amount of output produced by the
!                  program. IWORK(7) may have a value between 0 and 3.
!                  For IWORK(7) = 0 there is almost no output while for
!                  IWORK(7) = 3 the most output is produced.
!                  The default is 1.
!
!  IWORK(9)        Control of the Jacobian option specifying whether the user
!                  has supplied a Jacobian routine, or wants the program
!                  to approximate the Jacobian.
!                  0, the user has supplied the Jacobian.
!                  1, program is to use forward difference approximation.
!                  2, program is to use central difference approximation.
!                  IWORK(9) defaults to 0.
!
!  IWORK(10)       State indicator of the progress of the program.
!                  The values are:
!                  0, start up with unchecked starting point.
!                  1, first step.  Corrected starting point available.
!                  2, two successive continuation points available, as well
!                     as the tangent vector at the oldest of them.
!                  3, two successive continuation points available, as well
!                     as the tangent vector at the newest of them.
!
!  IWORK(11)       Index of the last computed target point. This is used to
!                  avoid repeated computation of a target point.  If a target
!                  point has been found, then the target index IWORK(5) is
!                  copied into IWORK(11).
!
!  IWORK(12)       Second best choice for the local parameterization index.
!                  This index may be tried if the first choice causes poor
!                  performance in the Newton corrector.
!
!  IWORK(13)       Beginning location in IWORK of unused integer work space
!                  available for use by the solver.
!
!  IWORK(14)       LIW, the user declared dimension of the array IWORK.
!
!  IWORK(15)       Beginning location in RWORK of unused real work space
!                  available for use by the solver.
!
!  IWORK(16)       LRW, the user declared dimension of RWORK.
!
!  IWORK(17)       Maximum number of corrector steps allowed during one run
!                  of the Newton process in which the Jacobian is updated at
!                  every step.  If the Jacobian is only evaluated at
!                  the beginning of the Newton iteration then 2*IWORK(17) steps
!                  are allowed.
!                  IWORK(17) must be greater than 0.  It defaults to 10.
!
!  IWORK(18)       Number of stepsize reductions that were needed for
!                  producing the last continuation point.
!
!  IWORK(19)       Total number of calls to the user Jacobian routine DF.
!
!  IWORK(20)       Total number of calls to the matrix factorization routine.
!                  If DENSLV is the chose solver then factorization is done by
!                  the LINPACK routine SGEFA.  If BANSLV is the solver, the
!                  LINPACK routine SGBFA will be used.
!
!  IWORK(21)       Total number of calls to the back-substitution routine.
!                  If DENSLV is the chosen solver, then back substitution is
!                  done by the LINPACK routine SGESL.  If BANSLV is used, then
!                  the LINPACK routine SGBSL will be used.
!
!  IWORK(22)       Total number of calls to the user function routine FX.
!
!  IWORK(23)       Total number of steps taken in limit point iterations.
!                  Each step involves determining an approximate limit point
!                  and applying a Newton iteration to correct it.
!
!  IWORK(24)       Total number of Newton corrector steps used during the
!                  computation of target points.
!
!  IWORK(25)       Total number of Newton steps taken during the correction
!                  of a starting point or the continuation points.
!
!  IWORK(26)       Total number of predictor stepsize-reductions needed
!                  since the start of the continuation procesds.
!
!  IWORK(27)       Total number of calls to the program.  This also
!                  corresponds to the number of points computed.
!
!  IWORK(28)       Total number of Newton steps taken during current iteration.
!
!  IWORK(30)       and on are reserved for use by the linear equation solver,
!                  and typically are used for pivoting.
!
!
!  H) The Real Work Array RWORK:
!
!  Input to PITCON includes the setting of some of the entries in RWORK.
!  Some of this input is optional.  The user input section of RWORK involves
!  entries 1 through 7 and possibly 18 and 20.
!
!  All entries of RWORK have default values.
!
!
!  RWORK(1)        ABSERR, absolute error tolerance.   This value is used
!                  mainly during the Newton iteration.  RWORK(1) defaults to
!                  SQRT(EPMACH) where EPMACH is the machine relative precision
!                  stored in RWORK(8).
!
!  RWORK(2)        RELERR, relative error tolerance.  This value is used mainly
!                  during the Newton iteration.  RWORK(2) defaults to
!                  SQRT(EPMACH) where EPMACH is the machine relative precision
!                  stored in RWORK(8).
!
!  RWORK(3)        HMIN, minimum allowable predictor stepsize.  If failures of
!                  the Newton correction force the stepsize down to this level,
!                  then the program will give up.  The default value is
!                  SQRT(EPMACH).
!
!  RWORK(4)        HMAX, maximum allowable predictor step.  Too generous a value
!                  may cause erratic behavior of the program.  The default
!                  value is SQRT(NVAR).
!
!  RWORK(5)        HTAN,  the predictor stepsize.  On first call, it should be
!                  set by the user.  Thereafter it is set by the program.
!                  RWORK(5) should be positive.  In order to travel in the
!                  negative direction, see RWORK(6).
!                  The default initial value equals 0.5*(RWORK(3)+RWORK(4)).
!
!  RWORK(6)        The local continuation direction, which is either +1.0
!                  or -1.0 .  This asserts that the program is moving in the
!                  direction of increasing or decreasing values of the local
!                  continuation variable, whose index is in IWORK(2).  On first
!                  call, the user must choose IWORK(2).  Therefore, by setting
!                  RWORK(6), the user may also specify whether the program is
!                  to move initially to increase or decrease the variable whose
!                  index is IWORK(2).
!                  RWORK(6) defaults to +1.
!
!  RWORK(7)        A target value.  It is only used if a target index
!                  has been specified through IWORK(5).  In that case, solution
!                  points with the IWORK(5) component equal to RWORK(7) are
!                  to be computed. The code will return each time it finds such
!                  a point.  RWORK(7) does not have a default value.  The
!                  program does not set it, and it is not referenced unless
!                  IWORK(5) has been set.
!
!  RWORK(8)        EPMACH, the value of the machine precision.  The computer
!                  can distinguish 1.0+EPMACH from 1.0, but it cannot
!                  distinguish 1.0+(EPMACH/2) from 1.0. This number is used
!                  when estimating a reasonable accuracy request on a given
!                  computer.  PITCON computes a value for EPMACH internally.
!
!  RWORK(9)        STEPX, the size, using the maximum-norm, of the last
!                  step of Newton correction used on the most recently
!                  computed point, whether a starting point, continuation
!                  point, limit point or target point.
!
!  RWORK(10)       A minimum angle used in the steplength computation,
!                  equal to 2.0*ARCCOS(1-EPMACH).
!
!  RWORK(11)       Estimate of the angle between the tangent vectors at the
!                  last two continuation points.
!
!  RWORK(12)       The pseudo-arclength coordinate of the previous continuation
!                  pointl; that is, the sum of the Euclidean distances between
!                  all computed continuation points beginning with the start
!                  point.  Thus each new point should have a larger coordinate,
!                  except for target and limit points which lie between the two
!                  most recent continuation points.
!
!  RWORK(13)       Estimate of the pseudo-arclength coordinate of the current
!                  continuation point.
!
!  RWORK(14)       Estimate of the pseudoarclength coordinate of the current
!                  limit or target point, if any.
!
!  RWORK(15)       Size of the correction of the most recent continuation
!                  point; that is, the maximum norm of the distance between the
!                  predicted point and the accepted corrected point.
!
!  RWORK(16)       Estimate of the curvature between the last two
!                  continuation points.
!
!  RWORK(17)       Sign of the determinant of the augmented matrix at the
!                  last continuation point whose tangent vector has been
!                  calculated.
!
!  RWORK(18)       This quantity is only used if the jacobian matrix is to
!                  be estimated using finite differences.  In that case,
!                  this value determines the size of the increments and
!                  decrements made to the current solution values, according
!                  to the following formula:
!
!                    Delta X(J) = RWORK(18) * (1.0 + ABS(X(J))).
!
!                  The value of every entry of the approximate jacobian could
!                  be extremely sensitive to RWORK(18).  Obviously, no value
!                  is perfect.  Values too small will surely cause singular
!                  jacobians, and values too large will surely cause inaccuracy.
!                  Little more is certain.  However, for many purposes, it
!                  is suitable to set RWORK(18) to the square root of the
!                  machine epsilon, or perhaps to the third or fourth root,
!                  if singularity seems to be occuring.
!
!                  RWORK(18) defaults to SQRT(SQRT(RWORK(8))).
!
!                  The user may set RWORK(18).  If it has a nonzero value on
!                  input, that value will be used.  Otherwise, the default
!                  value will be used.
!
!  RWORK(19)       Not currently used.
!
!  RWORK(20)       Maximum growth factor for the predictor stepsize based
!                  on the previous secant stepsize.  The stepsize algorithm
!                  will produce a suggested step that is no less that the
!                  previous secant step divided by this factor, and no greater
!                  than the previous secant step multiplied by that factor.
!                  RWORK(20) defaults to 3.
!
!  RWORK(21)       The (Euclidean) secant distance between the last two
!                  computed continuation points.
!
!  RWORK(22)       The previous value of RWORK(21).
!
!  RWORK(23)       A number judging the quality of the Newton corrector
!                  convergence at the last continuation point.
!
!  RWORK(24)       Value of the component of the current tangent vector
!                  corresponding to the current continuation index.
!
!  RWORK(25)       Value of the component of the previous tangent vector
!                  corresponding to the current continuation index.
!
!  RWORK(26)       Value of the component of the current tangent vector
!                  corresponding to the limit index in IWORK(6).
!
!  RWORK(27)       Value of the component of the previous tangent vector
!                  corresponding to the limit index in IWORK(6).
!
!  RWORK(28)       Value of RWORK(7) when the last target point was
!                  computed.
!
!  RWORK(29)       Sign of the determinant of the augmented matrix at the
!                  previous continuation point whose tangent vector has been
!                  calculated.
!
!  RWORK(30)       through RWORK(30+4*NVAR-1) are used by the program to hold
!                  an old and new continuation point, a tangent vector and a
!                  work vector.  Subsequent entries of RWORK are used by the
!                  linear solver.
!
!
!  I) Programming Notes:
!
!  The minimal input and user routines required to apply the program are:
!
!    Write a function routine FX of the form described above.
!    Use DENSLV as the linear equation solver by setting SLVNAM to DENSLV.
!    Skip writing a Jacobian routine by using the finite difference option.
!    Pass the name of FX as the Jacobian name as well.
!    Declare the name of the function FX as external.
!    Set NVAR in accordance with your problem.
!
!  Then:
!
!    Dimension the vector IWORK to the size LIW = 29+NVAR.
!    Dimension the vector RWORK to the size LRW = 29+NVAR*(NVAR+6).
!    Dimension IPAR(1) and FPAR(1) as required in the function routine.
!    Dimension XR(NVAR) and set it to an approximate solution of F(XR) = 0.
!
!  Set the work arrays as follows:
!
!    Initialize IWORK to 0 and RWORK to 0.0.
!
!    Set IWORK(1) = 0 (Problem startup)
!    Set IWORK(7) = 3 (Maximum internally generated output)
!    Set IWORK(9) = 1 (Forward difference Jacobian)
!
!  Now call the program repeatedly, and never change any of its arguments.
!  Check IERROR to decide whether the code is working satisfactorily.
!  Print out the vector XR to see the current solution point.
!
!
!  The most obvious input to try to set appropriately after some practice
!  would be the error tolerances RWORK(1) and RWORK(2), the minimum, maximum
!  and initial stepsizes RWORK(3), RWORK(4) and RWORK(5), and the initial
!  continuation index IWORK(2).
!
!  For speed and efficiency, a Jacobian routine should be written. It can be
!  checked by comparing its results with the finite difference Jacobian.
!
!  For a particular problem, the target and limit point input can be very
!  useful.  For instance, in the case of a discretized boundary value problem
!  with a real parameter it may be desirable to compare the computed solutions
!  for different discretization-dimensions and equal values of the parameter.
!  For this the target option can be used with the prescribed values of the
!  parameter. Limit points usually are of importance in connection with
!  stability considerations.
!
!
!  The routine REPS attempts to compute the machine precision, which in practice
!  is simply the smallest power of two that can be added to 1 to produce a
!  result greater than 1.  If the REPS routine misbehaves, you can replace
!  it by code that assigns a constant precomputed value, or by a call to
!  the PORT/SLATEC routine R1MACH.  REPS is called just once, in the
!  PITCON routine, and the value returned is stored into RWORK(8).
!
!  In subroutines DENSLV and BANSLV, the parameter "EPS" is set to RWORK(18)
!  and used in estimating jacobian matrices via finite differences.  If EPS is
!  too large, the jacobian will be very inaccurate.  Unfortunately, if EPS is
!  too small, the "finite" differences may become "infinitesmal" differences.
!  That is, the difference of two function values at close points may be
!  zero.  This is a very common problem, and occurs even with a function
!  like F(X) = X*X.  A singular jacobian is much worse than an inaccurate one,
!  so we have tried setting the default value of RWORK(18) to
!  SQRT(SQRT(RWORK(8)).  Such a value attempts to ward off singularity at the
!  expense of accuracy.  You may find for a particular problem or machine that
!  this value is too large and should be adjusted.  It is an utterly arbitrary
!  value.
!
!
!  J) Reference:
!
!  1.
!  Werner Rheinboldt,
!  Solution Field of Nonlinear Equations and Continuation Methods,
!  SIAM Journal of Numerical Analysis,
!  Volume 17, 1980, pages 221-237.
!
!  2.
!  Cor den Heijer and Werner Rheinboldt,
!  On Steplength Algorithms for a Class of Continuation Methods,
!  SIAM Journal of Numerical Analysis,
!  Volume 18, 1981, pages 925-947.
!
!  3.
!  Werner Rheinboldt,
!  Numerical Analysis of Parametrized Nonlinear Equations
!  John Wiley and Sons, New York, 1986
!
!  4.
!  Werner Rheinboldt and John Burkardt,
!  A Locally Parameterized Continuation Process,
!  ACM Transactions on Mathematical Software,
!  Volume 9, Number 2, June 1983, pages 215-235.
!
!  5.
!  Werner Rheinboldt and John Burkardt,
!  Algorithm 596, A Program for a Locally Parameterized Continuation Process,
!  ACM Transactions on Mathematical Software,
!  Volume 9, Number 2, June 1983, Pages 236-241.
!
!  6.
!  J J Dongarra, J R Bunch, C B Moler and G W Stewart,
!  LINPACK User's Guide,
!  Society for Industrial and Applied Mathematics,
!  Philadelphia, 1979.
!
!  7.
!  Richard Brent,
!  Algorithms for Minimization without Derivatives,
!  Prentice Hall, 1973.
!
!  8.
!  Tony Chan,
!  Deflated Decomposition of Solutions of Nearly Singular Systems,
!  Technical Report 225,
!  Computer Science Department,
!  Yale University,
!  New Haven, Connecticut, 06520,
!  1982.
!
!
!  L)  Sample programs:
!
!
!  A number of sample problems are included with PITCON.  There are several
!  examples of the Freudenstein-Roth function, which is a nice example
!  with only a few variables, and nice whole number starting and stopping
!  points.  Other sample problems demonstrate or test various options in
!  PITCON.
!
!
!  1:
!  pcprb1.f
!  The Freudenstein-Roth function.  3 variables.
!
!  This is a simple problem, whose solution curve has some severe bends.
!  This file solves the problem with a minimum of fuss.
!
!
!  2:
!  pcprb2.f
!  The Aircraft Stability problem.  8 variables.
!
!  This is a mildly nonlinear problem, whose solution curve has some
!  limit points that are difficult to track.
!
!
!  3:
!  pcprb3.f
!  A boundary value problem.  22 variables.
!
!  This problem has a limit point in the LAMBDA parameter, which we seek.  We
!  solve this problem 6 times, illustrating the use of full and banded
!  jacobians, and of user-generated, or forward or central difference
!  approximated jacobian matrices.
!
!  The first 21 variables represent the values of a function along a grid
!  of 21 points between 0 and 1.  By increasing the number of grid points,
!  the problem can be set up with arbitrarily many variables.  This change
!  requires changing a single parameter in the main program.
!
!
!  4:
!  pcprb4.f
!  The Freudenstein Roth function.  3 variables.
!
!  This version of the problem tests the parameterization fixing option.
!  The user may demand that PITCON always use a given index for continuation,
!  rather than varying the index from step to step.  The Freudenstein-Roth
!  curve may be parameterized by the variable of index 2, although this
!  may increase the number of steps required to traverse the curve.
!
!  This file carries out 6 runs, with and without the parameterization fixed,
!  and with no limit point search, or with limit points sought for index 1
!  or for index 3.
!
!
!  5:
!  pcprb5.f
!  A boundary value problem.  21 variables.
!
!  This problem has a limit point in the LAMBDA parameter, which we do not
!  seek.  We do seek target points where LAMBDA = 0.8, which occurs twice,
!  before and after LAMBDA "goes around the bend".
!
!  We run this test 3 times, comparing the behavior of the full Newton
!  iteration, the Newton iteration that fixes the Jacobian during the
!  correction of each point, and the Newton iteration that fixes the
!  Jacobian as long as possible.
!
!
!  6:
!  pcprb6.f
!  Freudenstein-Roth function.  3 variables.
!
!  This version of the problem demonstrates the jacobian checking option.
!  Two runs are made.  Each is allowed only five steps.  The first
!  run is with the correct jacobian.  The second run uses a defective
!  jacobian, and demonstrates not only the jacobian checker, but also
!  shows that "slightly" bad jacobians can cause the Newton convergence
!  to become linear rather than quadratic.
!
!
!  7:
!  pcprb7.f
!  The materially nonlinear problem, up to 71 variables.
!
!  This is a two point boundary value problem, representing the behavior
!  of a rod under a load.  The shape of the rod is represented by piecewise
!  polynomials, whose form and continuity are specifiable as options.
!
!  The problem as programmed can allow up to 71 variables, but a simple
!  change of a few parameters will allow the problem to be arbitrarily
!  large.
!
!  This is a complicated program.  It is intended to demonstrate the
!  advanced kinds of problems that can be solved with PITCON, as opposed
!  to the "toy" problems like the Freudenstein-Roth function.
!
!
!  8:
!  pcprb8.f
!  The Freudenstein-Roth function.  3 variables.
!
!  This version of the problem tests the jacobian approximation options.
!  We run the problem 3 times, first with the user jacobian, second with
!  the forward difference approximation, and third with the central
!  difference approximation.
!
!
!  Recent Changes:
!
!
!  Changes made for version 7.0:
!
!    Inserted simple versions of LAPACK routines.
!
!  Changes made for version 6.6:
!
!    Some calculations in COQUAL, for ITOP and IBOT, could involve integers
!    to negative powers.  These were replaced by real calculations for TOP
!    and BOT.
!
!    There is a stringent convergence test in CORRECTOR which severely slows
!    down the traversal of the Freudenstein Roth curve, because it forces
!    a last very small step, which causes the computation of the stepsize
!    to be skewed.  I temporarily turned this off.
!
!  Changes made for version 6.5:
!
!    Spun off "STAJAC" from START.
!
!    Code written in lower case.
!
!    Replaced all labeled DO statements with DO/ENDDO loops.
!
!  Changes made for version 6.4:
!
!    Calls to LINPACK replaced by calls to LAPACK
!    Added routines SGEDET and SGBDET to compute determinants.
!
!    Call to SGBTRF, SGBTRS, SGBDET had incorrect value of LDA,
!    which was corrected 21 January 1994.
!
!    Dropped LOUNIT.  All WRITE's are to unit * now.
!
!
!  Changes made for version 6.3:
!
!    27 July 1992: Printout of "Possible bifurcation" message will
!    now occur if IWRITE.GE.1.
!
!    HMIN was reset to MAX(HMIN,SQRT(EPS)) before.  Now it is only
!    set to SQRT(EPS) if HMIN is nonpositive.
!
!    30 September 1991: SKALE was not declared in LIMIT, causing
!    problems for the double precision code.
!
!    26 September 1991: corrected the formulation of the limit
!    point calculation.
!
!    12 September 1991: Jyun-Ming Chen reported that the program did not
!    actually fix the parameter to a user specified value when requested to do
!    so.  That is, when IWORK(3) = 1, the program is never supposed to alter the
!    input value of IWORK(2).  However, the program was doing so.  This error
!    has been corrected now.  The only exception occurs when the initial
!    input value of IWORK(2) is less than 1 or greater than NVAR, in which
!    case the program sets IWORK(2) to NVAR, no matter what the value
!    of IWORK(3).
!
!    The target and limit point calculations, and many other operations,
!    were "spun off" into subroutines.
!
!
!  Changes made for version 6.2:
!
!    The internal documentation was corrected.  It originally stated that
!    the user input portions of IWORK were entries 1 through 8, 17 and 29.
!    This was corrected to 1 through 9 and 17.
!
!    The entry RWORK(18) was previously unused.  It is now used to allow
!    the user to set the value of the finite difference increment used
!    for the approximation of the jacobian.
!
!    Modified DENSLV and BANSLV to allow more user control over the
!    Jacobian comparison or printout.  IWORK(1) may now have the values -1
!    through -10.  The user thus chooses to use forward or central
!    differences, and minimal or maximal printout.
!
!    Modified "Programming notes" section to explain the role of the
!    REPS routine, the new use of IWORK(1), and the new parameter RWORK(18).
!
!
!  Changes made for version 6.0:
!
!    1) When computing a starting point, the Newton convergence
!    criterion was relaxed to require only that the function
!    norm decrease, but not to require that the step norm
!    decrease.
!
!    2) In the Newton correction routine, the MAX norm was replaced
!    by the L2 norm when computing the size of the step and the
!    residual.  This was an attempt to control the 'jerkiness'
!    of the convergence for poorly scaled problems.
!
!    3) Added option to check Jacobian.
!
!    4) Apparently, a programming error in the previous version meant
!    that IWORK(29) was set to zero on startup.  This meant that the
!    program would not realize that the user was using a band storage
!    scheme.  This has been corrected.
!
!  Licensing:
!
!    This code is distributed under the GNU LGPL license. 
!
