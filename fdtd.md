## lumopt

## 学习

### fdtd仿真全流程

1. 设计基模板:base_script
   1. 使用FDTD内嵌.$lsf$脚本编写
   2. 或者调用$lumapi$编写.$py$脚本
2. 编写优化程序
   1. 调用$lumopt$编写

### 原理(adjoint method，伴随状态方法)

fdtd中问题通常都可以简化为
$$
A(p)x =b\tag1
$$

- $A$表示的是麦克斯韦方程组，以及空间中每个离散化点下层结构的材料介电常数(permittivity)和磁导率(permeability)
- $x$表示模拟区域中的所有字段
- $b$表示自由电流或等效的源

$$
F(p)=F(x(p))
$$

梯度(Gradient)求法
$$
\frac{\partial F}{\partial p} = \frac{\partial F}{\partial x}\frac{\partial x}{\partial p}
$$
如果已知$FOM$也就是$F$跟随场(filed)$x$的变化即${\partial F}/{\partial x}$，以及$x$跟随场中各个参数$p$的变化即$\partial x / \partial p$，就能求得梯度，但要求解$\partial x / \partial p$首先需要回到控制方程组$(1)$，对两边求偏导
$$
A\frac{\partial x}{\partial p} = \frac{\partial b}{\partial p}-\frac{\partial A}{\partial p}x
$$

- $\partial b / \partial p$：照度随参数$p$的变化而变化(因为$b$表示光源)
- $(\partial A / \partial p)x$：几何形状随参数$p$的变化而变化

这个表达$(\partial x/ \partial p)x$实际上并没有降低我们的计算成本，加入向量$v^T$简化方程使标量相等
$$
v^T(A\frac{\partial x}{\partial p}) = v^T(\frac{\partial b}{\partial p}-\frac{\partial A}{\partial p}x)
$$
这简化了问题，但我们仍然需要指定这个向量$v^T$.如果我们要求$v^TA$=$\frac{\partial F}{\partial x}$我们要会立即得到一个设计灵敏度。
$$
\frac{\partial F}{\partial p} = \frac{\partial F}{\partial x}\frac{\partial x}{\partial p}=v^T(A\frac{\partial x}{\partial p}) = v^T(\frac{\partial b}{\partial p}-\frac{\partial A}{\partial p}x)
\\finally
\\\frac{\partial F}{\partial p} = v^T(\frac{\partial b}{\partial p}-\frac{\partial A}{\partial p}x)
$$
上述表达式的含义：使用两个模拟来计算梯度。

- 正向模拟中：$x$由于源$b$变化而变化，在存在可优化几何形状的情况下，使用通过投射到指定模式下的视场轮廓上的孔径的功率来表示$FOM$

- 伴随模拟中：源b替换为伴随源(adjoint source)，取代了FOM，生成的字段再次记录在优化区域的所有位置以进行计算$v$可以将其概念化为时间反转模拟，以找到伴随的麦克斯韦方程组。
  $$
  A^Tv=\frac{\partial F^T}{\partial x}
  $$
  

#### figures_of_merit

##### modematch

###### \__init__

- $target\_T_{fwd}\_weights$ = lambda wl: np.ones(wl.size)表示11个取样点的权重，默认都为1

  shape:[11, 1] [1, 1, 1, ..., 1, 1]

- 例如：$wavelengths$：波长，假设选取了11个波长点，shape:[11, 1] [400, 430, 460, ..., 670, 700]

- $T\_fwd\_vs\_wavelength$: 波长取样点对应的透射率，shape:[11, 1] [0.659, 0.66,...., 0.346]

  

```python
def __init__(self, monitor_name, mode_number, direction, multi_freq_src = False, target_T_fwd = lambda wl: np.ones(wl.size), norm_p = 1, target_fom = 0, target_T_fwd_weights = lambda wl: np.ones(wl.size), rotation_angle_theta = 0, rotation_angle_phi = 0, rotation_offset = 0):
    self.monitor_name = str(monitor_name)
    if not self.monitor_name:
        raise UserWarning('empty monitor name.')
    self.mode_expansion_monitor_name = monitor_name + '_mode_exp'
    self.adjoint_source_name = monitor_name + '_mode_src'
    self.mode_number = mode_number
    self.target_fom = target_fom

    self.rotation_angle_theta = rotation_angle_theta   #*3.14/180
    self.rotation_angle_phi = rotation_angle_phi
    self.rotation_offset = rotation_offset
    
    
    if is_int(mode_number):
        self.mode_number = int(mode_number)
        if self.mode_number <= 0:
         raise UserWarning('mode number should be positive.')
    else:
        self.mode_number = mode_number
    self.direction = str(direction)
    self.multi_freq_src = bool(multi_freq_src)
    if self.direction != 'Forward' and self.direction != 'Backward':
        raise UserWarning('invalid propagation direction.')
    target_T_fwd_result = target_T_fwd(np.linspace(0.1e-6, 10.0e-6, 1000))
    
    if target_T_fwd_result.size != 1000:
        raise UserWarning('target transmission must return a flat vector with the requested number of wavelength samples.')
    elif np.any(target_T_fwd_result.min() < 0.0) or np.any(target_T_fwd_result.max() > 1.0):
        raise UserWarning('target transmission must always return numbers between zero and one.')
    else:
        self.target_T_fwd = target_T_fwd
        
    target_T_fwd_weights_result = target_T_fwd_weights(np.linspace(0.1e-6, 10.0e-6, 1000))
    
    if target_T_fwd_weights_result.size != 1000:
        raise UserWarning('target transmission weights must return a flat vector with the requested number of wavelength samples.')
    elif np.any(target_T_fwd_weights_result.min() < 0.0):
        raise UserWarning('target transmission weights must always return positive numbers or zero.')
    else:
        self.target_T_fwd_weights = target_T_fwd_weights
    self.norm_p = int(norm_p)
    if self.norm_p < 1:
        raise UserWarning('exponent p for norm must be positive.')
```

###### initialize

```python
def initialize(self, sim):
    self.check_monitor_alignment(sim)
    
# 添加ModeExpansionMonitor: fom_mode_exp
# 根据ModeMatch中的
#self.monitor_name, self.mode_expansion_monitor_name, self.mode_number, self.rotation_angle_theta, self.rotation_angle_phi, self.rotation_offset
    ModeMatch.add_mode_expansion_monitor(sim, self.monitor_name, self.mode_expansion_monitor_name, self.mode_number, self.rotation_angle_theta, self.rotation_angle_phi, self.rotation_offset)
    
    
    adjoint_injection_direction = 'Backward' if self.direction == 'Forward' else 'Forward'
    # 初始化ModeSource：fom_mode_src（根据的是fom）
    # self.monitor_name = 'fom'
    # self.mode_number = MatchMode中的
    # adjoint_source_name = 'fom_mode_src'
    ModeMatch.add_mode_source(sim, self.monitor_name, self.adjoint_source_name, adjoint_injection_direction, self.mode_number, self.multi_freq_src, self.rotation_angle_theta, self.rotation_angle_phi, self.rotation_offset)
```

###### fom_wavelength_integral

根据$功率-波长曲线图$进行品质因子FOM(figure of merit)
$$
F=\left (\frac{1}{\lambda_2-\lambda_1} \int_{\lambda_1}^{\lambda_2} {|T_0(\lambda)|^p\,d\lambda} \right)^{1/p}- \left (\frac{1}{\lambda_2-\lambda_1}\int_{\lambda1}^{\lambda 2}|T(\lambda)-T_0(\lambda)|^p \, d\lambda\right)^{1/p}
$$

- $T_0(\lambda)$对应$ModeMatch$设置中的$target\_T\_fwd$，也就是目标透射强度
- $T(\lambda)$对应实际测得的功率（透射强度）
- 注意上面公式默认了取得波长离散点的权重均为1，故没有显示

```python
@staticmethod
# wavelengths：波长，假设选取了11个波长点，shape:[11, 1] [400, 430, 460, ..., 670, 700]
# T_fwd_vs_wavelength: 波长取样点对应的透射率，shape:[11, 1] [0.659, 0.66,...., 0.346]

def fom_wavelength_integral(T_fwd_vs_wavelength, wavelengths, target_T_fwd, norm_p, target_T_fwd_weights):
    target_T_fwd_vs_wavelength = target_T_fwd(wavelengths).flatten()
    target_T_fwd_weights_vs_wavelength = target_T_fwd_weights(wavelengths).flatten()
    if len(wavelengths) > 1:
        wavelength_range = wavelengths.max() - wavelengths.min()
        assert wavelength_range > 0.0, "wavelength range must be positive."
        
        # 计算积分：也就是λ×T0(λ)的p次方，此处p选为1
        # 再除以波长范围（λ2-λ1）
        # T_fwd_integrand:乘了波长权重target_T_fwd_weights_vs_wavelength的离散值
        T_fwd_integrand = np.multiply(target_T_fwd_weights_vs_wavelength, np.power(np.abs(target_T_fwd_vs_wavelength), norm_p)) / wavelength_range #Include Here
        #T_fwd_integrand = np.power(np.abs(target_T_fwd_vs_wavelength), norm_p) / wavelength_range
         # 最后shape为[11, 1]的T_fwd_integrand进行积分后，再开1/p次方得到了第一个数const_term
        const_term = np.power(np.trapz(y = T_fwd_integrand, x = wavelengths), 1.0 / norm_p)
        
        # T_fwd_vs_wavelength：也就是T(λ)，实际测得的透射强度
        T_fwd_error = np.abs(T_fwd_vs_wavelength.flatten() - target_T_fwd_vs_wavelength)
        # T_fwd_error_integrand：同样乘了波长权重
        T_fwd_error_integrand = np.multiply(target_T_fwd__vs_wavelength, np.power(T_fwd_error, norm_p)) / wavelength_range #Include here
        #T_fwd_error_integrand = np.power(T_fwd_error, norm_p) / wavelength_range
        error_term = np.power(np.trapz(y = T_fwd_error_integrand, x = wavelengths), 1.0 / norm_p)
        fom = const_term - error_term
    else:
        fom = np.abs(target_T_fwd_vs_wavelength) - np.abs(T_fwd_vs_wavelength.flatten() - target_T_fwd_vs_wavelength)
        
    # 只取透射强度的实数部分
    return fom.real
```

###### add_mode_source

```python
# 初始化ModeSource：fom_mode_src（根据的是fom）
def add_mode_source(sim, monitor_name, source_name, direction, mode, multi_freq_src, rotation_angle_theta, rotation_angle_phi, rotation_offset):
    if sim.fdtd.getnamednumber('FDTD') == 1:
        sim.fdtd.addmode()
    elif sim.fdtd.getnamednumber('varFDTD') == 1:
        sim.fdtd.addmodesource()
    else:
        raise UserWarning('no FDTD or varFDTD solver object could be found.')
    sim.fdtd.set('name', source_name)
    sim.fdtd.select(source_name)
    monitor_type = sim.fdtd.getnamed(monitor_name, 'monitor type')
    geo_props, normal = ModeMatch.cross_section_monitor_props(monitor_type)
    sim.fdtd.setnamed(source_name, 'injection axis', normal.lower() + '-axis')
    if sim.fdtd.getnamednumber('varFDTD') == 1:
        geo_props.remove('z')
    for prop_name in geo_props:
        prop_val = sim.fdtd.getnamed(monitor_name, prop_name)
        sim.fdtd.setnamed(source_name, prop_name, prop_val)
    sim.fdtd.setnamed(source_name, 'override global source settings', False)
    sim.fdtd.setnamed(source_name, 'direction', direction)
    if sim.fdtd.haveproperty('multifrequency mode calculation'):
        sim.fdtd.setnamed(source_name, 'multifrequency mode calculation', multi_freq_src)
        if multi_freq_src:
            sim.fdtd.setnamed(source_name, 'frequency points', sim.fdtd.getglobalmonitor('frequency points'))
    
    if is_int(mode):
        sim.fdtd.setnamed(source_name, 'mode selection', 'user select')
        sim.fdtd.select(source_name)
        sim.fdtd.updatesourcemode(int(mode))
    else:
        sim.fdtd.setnamed(source_name, 'mode selection', mode)
        sim.fdtd.select(source_name)
        sim.fdtd.updatesourcemode()

    # passing tilting properties
    sim.fdtd.setnamed(source_name, 'theta', rotation_angle_theta)
    if sim.fdtd.getnamednumber('FDTD') == 1:
            if sim.fdtd.getnamed('FDTD', 'dimension') == '3D':  
                sim.fdtd.setnamed(source_name, 'phi', rotation_angle_phi)
    sim.fdtd.setnamed(source_name, 'rotation offset', rotation_offset)
```

#### optimizers

##### generic optimizers

```python
class ScipyOptimizers(Minimizer):
    def __init__(self, max_iter, method = 'L-BFGS-B', scaling_factor = None, pgtol = 1.0e-5, ftol = 1.0e-12, scale_initial_gradient_to = 0, penalty_fun = None, penalty_jac = None):
        super(ScipyOptimizers,self).__init__(max_iter = max_iter,
                                             scaling_factor = scaling_factor,
                                             scale_initial_gradient_to = scale_initial_gradient_to,
                                             penalty_fun = penalty_fun,
                                             penalty_jac = penalty_jac)
        self.method = str(method)
        self.pgtol = float(pgtol)
        self.ftol=float(ftol)
       
    # 
    def run(self):
        print('Running scipy optimizer')
        print('bounds = {}'.format(self.bounds))
        print('start = {}'.format(self.start_point))
        res = spo.minimize(fun = self.callable_fom,
                           x0 = self.start_point,
                           jac = self.callable_jac,
                           bounds = self.bounds,
                           callback = self.callback,
                           options = {'maxiter':self.max_iter, 'disp':True, 'gtol':self.pgtol,'ftol':self.ftol},
                           method = self.method)
        # res.x /= self.scaling_factor
        res.x = res.x /self.scaling_factor + self.scaling_offset
        res.fun = -res.fun
        if hasattr(res, 'jac'):
            res.jac = -res.jac*self.scaling_factor
        print('Number of FOM evaluations: {}'.format(res.nit))
        print('FINAL FOM = {}'.format(res.fun))
        print('FINAL PARAMETERS = {}'.format(res.x))
        return res
```

#### scipy-optimize-_minimize

##### minimize

[scipy.optimize.minimize — SciPy v1.11.4 Manual](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html#scipy.optimize.minimize)

```python
def minimize(fun, x0, args=(), method=None, jac=None, hess=None,
             hessp=None, bounds=None, constraints=(), tol=None,
             callback=None, options=None):
    """Minimization of scalar function of one or more variables.

    Parameters
    ----------
    fun : callable
        The objective function to be minimized.

            ``fun(x, *args) -> float``

        where x is an 1-D array with shape (n,) and `args`
        is a tuple of the fixed parameters needed to completely
        specify the function.
    x0 : ndarray, shape (n,)
        Initial guess. Array of real elements of size (n,),
        where 'n' is the number of independent variables.
    args : tuple, optional
        Extra arguments passed to the objective function and its
        derivatives (`fun`, `jac` and `hess` functions).
    method : str or callable, optional
        Type of solver.  Should be one of

            - 'Nelder-Mead' :ref:`(see here) <optimize.minimize-neldermead>`
            - 'Powell'      :ref:`(see here) <optimize.minimize-powell>`
            - 'CG'          :ref:`(see here) <optimize.minimize-cg>`
            - 'BFGS'        :ref:`(see here) <optimize.minimize-bfgs>`
            - 'Newton-CG'   :ref:`(see here) <optimize.minimize-newtoncg>`
            - 'L-BFGS-B'    :ref:`(see here) <optimize.minimize-lbfgsb>`
            - 'TNC'         :ref:`(see here) <optimize.minimize-tnc>`
            - 'COBYLA'      :ref:`(see here) <optimize.minimize-cobyla>`
            - 'SLSQP'       :ref:`(see here) <optimize.minimize-slsqp>`
            - 'trust-constr':ref:`(see here) <optimize.minimize-trustconstr>`
            - 'dogleg'      :ref:`(see here) <optimize.minimize-dogleg>`
            - 'trust-ncg'   :ref:`(see here) <optimize.minimize-trustncg>`
            - 'trust-exact' :ref:`(see here) <optimize.minimize-trustexact>`
            - 'trust-krylov' :ref:`(see here) <optimize.minimize-trustkrylov>`
            - custom - a callable object (added in version 0.14.0),
              see below for description.

        If not given, chosen to be one of ``BFGS``, ``L-BFGS-B``, ``SLSQP``,
        depending if the problem has constraints or bounds.
    jac : {callable,  '2-point', '3-point', 'cs', bool}, optional
        Method for computing the gradient vector. Only for CG, BFGS,
        Newton-CG, L-BFGS-B, TNC, SLSQP, dogleg, trust-ncg, trust-krylov,
        trust-exact and trust-constr. If it is a callable, it should be a
        function that returns the gradient vector:

            ``jac(x, *args) -> array_like, shape (n,)``

        where x is an array with shape (n,) and `args` is a tuple with
        the fixed parameters. Alternatively, the keywords
        {'2-point', '3-point', 'cs'} select a finite
        difference scheme for numerical estimation of the gradient. Options
        '3-point' and 'cs' are available only to 'trust-constr'.
        If `jac` is a Boolean and is True, `fun` is assumed to return the
        gradient along with the objective function. If False, the gradient
        will be estimated using '2-point' finite difference estimation.
    hess : {callable, '2-point', '3-point', 'cs', HessianUpdateStrategy},  optional
        Method for computing the Hessian matrix. Only for Newton-CG, dogleg,
        trust-ncg,  trust-krylov, trust-exact and trust-constr. If it is
        callable, it should return the  Hessian matrix:

            ``hess(x, *args) -> {LinearOperator, spmatrix, array}, (n, n)``

        where x is a (n,) ndarray and `args` is a tuple with the fixed
        parameters. LinearOperator and sparse matrix returns are
        allowed only for 'trust-constr' method. Alternatively, the keywords
        {'2-point', '3-point', 'cs'} select a finite difference scheme
        for numerical estimation. Or, objects implementing
        `HessianUpdateStrategy` interface can be used to approximate
        the Hessian. Available quasi-Newton methods implementing
        this interface are:

            - `BFGS`;
            - `SR1`.

        Whenever the gradient is estimated via finite-differences,
        the Hessian cannot be estimated with options
        {'2-point', '3-point', 'cs'} and needs to be
        estimated using one of the quasi-Newton strategies.
        Finite-difference options {'2-point', '3-point', 'cs'} and
        `HessianUpdateStrategy` are available only for 'trust-constr' method.
    hessp : callable, optional
        Hessian of objective function times an arbitrary vector p. Only for
        Newton-CG, trust-ncg, trust-krylov, trust-constr.
        Only one of `hessp` or `hess` needs to be given.  If `hess` is
        provided, then `hessp` will be ignored.  `hessp` must compute the
        Hessian times an arbitrary vector:

            ``hessp(x, p, *args) ->  ndarray shape (n,)``

        where x is a (n,) ndarray, p is an arbitrary vector with
        dimension (n,) and `args` is a tuple with the fixed
        parameters.
    bounds : sequence or `Bounds`, optional
        Bounds on variables for L-BFGS-B, TNC, SLSQP and
        trust-constr methods. There are two ways to specify the bounds:

            1. Instance of `Bounds` class.
            2. Sequence of ``(min, max)`` pairs for each element in `x`. None
               is used to specify no bound.

    constraints : {Constraint, dict} or List of {Constraint, dict}, optional
        Constraints definition (only for COBYLA, SLSQP and trust-constr).
        Constraints for 'trust-constr' are defined as a single object or a
        list of objects specifying constraints to the optimization problem.
        Available constraints are:

            - `LinearConstraint`
            - `NonlinearConstraint`

        Constraints for COBYLA, SLSQP are defined as a list of dictionaries.
        Each dictionary with fields:

            type : str
                Constraint type: 'eq' for equality, 'ineq' for inequality.
            fun : callable
                The function defining the constraint.
            jac : callable, optional
                The Jacobian of `fun` (only for SLSQP).
            args : sequence, optional
                Extra arguments to be passed to the function and Jacobian.

        Equality constraint means that the constraint function result is to
        be zero whereas inequality means that it is to be non-negative.
        Note that COBYLA only supports inequality constraints.
    tol : float, optional
        Tolerance for termination. For detailed control, use solver-specific
        options.
    options : dict, optional
        A dictionary of solver options. All methods accept the following
        generic options:

            maxiter : int
                Maximum number of iterations to perform. Depending on the
                method each iteration may use several function evaluations.
            disp : bool
                Set to True to print convergence messages.
```

#### scipy-optimize-lbfgsb

##### _minimize_lbfgsb

```python
def _minimize_lbfgsb(fun, x0, args=(), jac=None, bounds=None,
                     disp=None, maxcor=10, ftol=2.2204460492503131e-09,
                     gtol=1e-5, eps=1e-8, maxfun=15000, maxiter=15000,
                     iprint=-1, callback=None, maxls=20, **unknown_options):

    _check_unknown_options(unknown_options)
    m = maxcor
    epsilon = eps
    pgtol = gtol
    factr = ftol / np.finfo(float).eps

    x0 = asarray(x0).ravel()
    n, = x0.shape

    if bounds is None:
        bounds = [(None, None)] * n
    if len(bounds) != n:
        raise ValueError('length of x0 != length of bounds')
    # unbounded variables must use None, not +-inf, for optimizer to work properly
    bounds = [(None if l == -np.inf else l, None if u == np.inf else u) for l, u in bounds]

    if disp is not None:
        if disp == 0:
            iprint = -1
        else:
            iprint = disp

    n_function_evals, fun = wrap_function(fun, ())
    if jac is None:
        def func_and_grad(x):
            f = fun(x, *args)
            g = _approx_fprime_helper(x, fun, epsilon, args=args, f0=f)
            return f, g
    else:
        def func_and_grad(x):
            f = fun(x, *args)
            g = jac(x, *args)
            return f, g

    nbd = zeros(n, int32)
    low_bnd = zeros(n, float64)
    upper_bnd = zeros(n, float64)
    bounds_map = {(None, None): 0,
                  (1, None): 1,
                  (1, 1): 2,
                  (None, 1): 3}
    for i in range(0, n):
        l, u = bounds[i]
        if l is not None:
            low_bnd[i] = l
            l = 1
        if u is not None:
            upper_bnd[i] = u
            u = 1
        nbd[i] = bounds_map[l, u]

    if not maxls > 0:
        raise ValueError('maxls must be positive.')

    x = array(x0, float64)
    f = array(0.0, float64)
    g = zeros((n,), float64)
    wa = zeros(2*m*n + 5*n + 11*m*m + 8*m, float64)
    iwa = zeros(3*n, int32)
    task = zeros(1, 'S60')
    csave = zeros(1, 'S60')
    lsave = zeros(4, int32)
    isave = zeros(44, int32)
    dsave = zeros(29, float64)

    task[:] = 'START'

    n_iterations = 0

    while 1:
        # x, f, g, wa, iwa, task, csave, lsave, isave, dsave = \
        _lbfgsb.setulb(m, x, low_bnd, upper_bnd, nbd, f, g, factr,
                       pgtol, wa, iwa, task, iprint, csave, lsave,
                       isave, dsave, maxls)
        task_str = task.tostring()
        if task_str.startswith(b'FG'):
            # The minimization routine wants f and g at the current x.
            # Note that interruptions due to maxfun are postponed
            # until the completion of the current minimization iteration.
            # Overwrite f and g:
            f, g = func_and_grad(x)
        elif task_str.startswith(b'NEW_X'):
            # new iteration
            n_iterations += 1
            if callback is not None:
                callback(np.copy(x))

            if n_iterations >= maxiter:
                task[:] = 'STOP: TOTAL NO. of ITERATIONS REACHED LIMIT'
            elif n_function_evals[0] > maxfun:
                task[:] = ('STOP: TOTAL NO. of f AND g EVALUATIONS '
                           'EXCEEDS LIMIT')
        else:
            break

    task_str = task.tostring().strip(b'\x00').strip()
    if task_str.startswith(b'CONV'):
        warnflag = 0
    elif n_function_evals[0] > maxfun or n_iterations >= maxiter:
        warnflag = 1
    else:
        warnflag = 2

    # These two portions of the workspace are described in the mainlb
    # subroutine in lbfgsb.f. See line 363.
    s = wa[0: m*n].reshape(m, n)
    y = wa[m*n: 2*m*n].reshape(m, n)

    # See lbfgsb.f line 160 for this portion of the workspace.
    # isave(31) = the total number of BFGS updates prior the current iteration;
    n_bfgs_updates = isave[30]

    n_corrs = min(n_bfgs_updates, maxcor)
    hess_inv = LbfgsInvHessProduct(s[:n_corrs], y[:n_corrs])

    return OptimizeResult(fun=f, jac=g, nfev=n_function_evals[0],
                          nit=n_iterations, status=warnflag, message=task_str,
                          x=x, success=(warnflag == 0), hess_inv=hess_inv)
```

#### optimization

##### callable_fom

求解fom值

```python 
def callable_fom(self, params):

    self.sim.fdtd.clearjobs()
    iter = self.optimizer.iteration if self.store_all_simulations else 0
    print('Making forward solve')
    forward_job_name = self.make_forward_sim(params, iter)
    self.sim.fdtd.addjob(forward_job_name)
    if self.optimizer.concurrent_adjoint_solves():
        print('Making adjoint solve')
        adjoint_job_name = self.make_adjoint_sim(params, iter)
        self.sim.fdtd.addjob(adjoint_job_name)
    print('Running solves')
    self.sim.fdtd.runjobs()

    print('Processing forward solve')
    fom = self.process_forward_sim(iter)

    ## If the geometry class has an additional penalty term (e.g. min feature size for topology)
    if hasattr(self.geometry,'calc_penalty_term'):
        fom_penalty = self.geometry.calc_penalty_term(self.sim, params)
        print('Actual fom: {}, Penalty term: {}, Total fom: {}'.format(fom, fom_penalty,(fom + fom_penalty)))
        fom += fom_penalty

    return fom
```

```python
# FDTD model
# sim:fdtd实例
self.sim = Simulation(self.workingDir, self.use_var_fdtd, self.hide_fdtd_cad)

# 初始化基底
self.base_script(self.sim.fdtd)
# 设置全局波长
Optimization.set_global_wavelength(self.sim, self.wavelengths)
# 设置局部光源波长
Optimization.set_source_wavelength(self.sim, self.source_name, self.fom.multi_freq_src, len(self.wavelengths))
# 对局部光源进行参数设置
self.sim.fdtd.setnamed('opt_fields', 'override global monitor settings', False)
self.sim.fdtd.setnamed('opt_fields', 'spatial interpolation', 'none')
# 初始化Index Monitor:opt_fileds_index
# 用来记录折射率-波长曲线，观测折射率是否变化
Optimization.add_index_monitor(self.sim, 'opt_fields', self.wavelengths)

if self.use_deps:
    Optimization.set_use_legacy_conformal_interface_detection(self.sim, False)

# Optimizer
start_params = self.geometry.get_current_params()

# 初始化待优化区域：也就是Optimization中的gometry
self.geometry.add_geo(self.sim, start_params, only_update = False)

# If we don't have initial parameters yet, try to extract them from the simulation (this is mostly for topology optimization)
if start_params is None:
    self.geometry.extract_parameters_from_simulation(self.sim)
    start_params = self.geometry.get_current_params()

callable_fom = self.callable_fom
callable_jac = self.callable_jac
bounds = self.geometry.bounds

# 初始化ModeExpansionMonitor:fom_mode_exp
# 用于观测fom中的指定偏振状态
# 初始化ModeSource：fom_mode_src
# 用于替代原有的光源
# 此处对应ModeMatch
self.fom.initialize(self.sim)

# 初始化优化器optimizer
self.optimizer.initialize(start_params = start_params, callable_fom = callable_fom, callable_jac = callable_jac, bounds = bounds, plotting_function = plotting_function_fwd)

# 至此，所有初始化工作已经完成
```

##### calculate_gradients

用于计算梯度，使用的方法跟$optimization$初始化中的$use\_deps$有关

- $use\_deps=True$，则使用常规的从meshing中进行磁介质微分计算的方法
- $use\_deps=False$，则使用形状导数近似法（来源：Owen Miller）

```python
def calculate_gradients(self):
    """ Calculates the gradient of the figure of merit (FOM) with respect to each of the optimization parameters.
        It assumes that both the forward and adjoint solves have been run so that all the necessary field results
        have been collected. There are currently two methods to compute the gradient:
            1) using the permittivity derivatives calculated directly from meshing (use_deps == True) and
            2) using the shape derivative approximation described in Owen Miller's thesis (use_deps == False).
    """
```

##### Optimization

```python
opt = Optimization(base_script = y_branch_base,
                  wavelengths = wavelengths,
                  fom = fom,
                  geometry = polygon,
                  optimizer = optimizer,
                  use_var_fdtd = True,
                  hide_fdtd_cad = False,
                  use_deps = True,
                  plot_history = True,
                  store_all_simulations = False)
```

base_script: 基本结构（毛胚房）

fom: 优化的**目标函数**

geometry：优化的目标区域

hide_fdtd_cad：隐藏fdtd变化开关

plot_history：是否绘图

![image-20240115155657302](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240115155657302.png)

##### run

```python
def run(self, working_dir = None):
    # 初始化所有的elements(rectangle, source, mesh,...)
    self.initialize(working_dir)
    
    # 初始化matplotlib
    self.init_plotter()
    if self.plotter.movie:
        with self.plotter.writer.saving(self.plotter.fig, os.path.join(self.workingDir,'optimization.png'), 100):
            
            # lumopt，启动！
            self.optimizer.run()
    else:
        self.optimizer.run()
```

#### gometries

##### gometry

###### d_eps_on_cad_serial

#### utilities

##### base_script

```python
""" Copyright chriskeraly
    Copyright (c) 2019 Lumerical Inc. """

import os
from inspect import signature

from lumapi import FDTD
from lumapi import MODE
from lumopt.utilities.load_lumerical_scripts import load_from_lsf

class BaseScript(object):
    """ 
        Proxy class for creating a base simulation. It acts as an interface to place the appropriate call in the FDTD CAD
        to build the base simulation depending on the input object. Options are:
            1) a Python callable,
            2) any visible *.fsp project file,
            3) any visible *.lsf script file or
            4) a plain string with a Lumerical script.
        
        Parameters:
        -----------
        :script_obj: executable, file name or plain string.
    """

    def __init__(self, script_obj):
        if callable(script_obj):
            self.callable_obj = script_obj
            params = signature(script_obj).parameters
            if len(params) > 1:
                raise UserWarning('function to create base simulation must take a single argument (handle to FDTD CAD).')
        elif isinstance(script_obj, str):
            if '.fsp' in script_obj and os.path.isfile(script_obj) or '.lms' in script_obj and os.path.isfile(script_obj):
                self.project_file = os.path.abspath(script_obj)
            elif '.lsf' in script_obj and os.path.isfile(script_obj):
                self.script_str = load_from_lsf(os.path.abspath(script_obj))
            else:
                self.script_str = str(script_obj)
        else:
            raise UserWarning('object for generating base simulation must be a Python function, a file name or a string with a Lumerical script.')

    def __call__(self, cad_handle):
        return self.eval(cad_handle)

    def eval(self, cad_handle):
        if not isinstance(cad_handle, FDTD) and not isinstance(cad_handle, MODE):
            raise UserWarning('input must be handle returned by lumapi.FDTD.')
        if hasattr(self, 'callable_obj'):
            return self.callable_obj(cad_handle)
        elif hasattr(self, 'project_file'):
            return cad_handle.load(self.project_file)
        elif hasattr(self, 'script_str'):
            return cad_handle.eval(self.script_str)
        else:
            raise RuntimeError('un-initialized object.')
            
    def __eq__(self, other):
        if hasattr(self, 'callable_obj') and hasattr(other, 'callable_obj'):
            return self.callable_obj == other.callable_obj 
        elif hasattr(self, 'project_file') and hasattr(other, 'project_file'):
            return self.project_file == other.project_file 
        elif hasattr(self, 'script_str') and hasattr(other, 'script_str'):
            return self.script_str == other.script_str 
        else:
            False
```

