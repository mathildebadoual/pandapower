﻿.. _trafo:

=============
Transformer
=============

.. |br| raw:: html

    <br>
    
.. seealso::

    :ref:`Unit Systems and Conventions <conventions>` |br|
    :ref:`Standard Type Libraries <std_types>`

Create Function
=====================

Transformers can be either created from the standard type library (create_transformer) or with custom values (create_transformer_from_parameters).

.. autofunction:: pandapower.create_transformer

.. autofunction:: pandapower.create_transformer_from_parameters


Input Parameters
=====================

*net.trafo*

.. tabularcolumns:: |p{0.15\linewidth}|p{0.1\linewidth}|p{0.25\linewidth}|p{0.4\linewidth}|
.. csv-table:: 
   :file: trafo_par.csv
   :delim: ;
   :widths: 15, 10, 25, 40
  
\*necessary for executing a power flow calculation |br| \*\*optimal power flow parameter

.. note:: The transformer loading constraint for the optimal power flow corresponds to the option trafo_loading="current":

   
Electric Model
=================

The equivalent circuit used for the transformer can be set in the power flow with the parameter "trafo_model".
   
*trafo_model='t':*

.. image:: trafo_t.png
	:width: 30em
	:align: center

*trafo_model='pi':*

.. image:: trafo_pi.png
	:width: 30em
	:align: center

    
Transformer Ratio
-------------------

The magnitude of the transformer ratio is given as:

.. math::
   :nowrap:
   
   \begin{align*}
   n &= \frac{V_{ref, HV, transformer}}{V_{ref, LV, transformer}} \cdot \frac{V_{ref, LV bus}}{V_{ref, HV bus}}
   \end{align*}

The reference voltages of the high- and low voltage buses are taken from the net.bus table. The reference voltage of the transformer  is taken directly from the transformer table:

.. math::
   :nowrap:
   
   \begin{align*}
    V_{ref, HV, transformer} &= vn\_hv\_kv \\
    V_{ref, LV, transformer} &= vn\_lv\_kv
   \end{align*}
   

If the power flow is run with voltage_angles=True, the complex ratio is given as:

.. math::
   :nowrap:
   
   \begin{align*}
   \underline{n} &= n \cdot e^{j \cdot \theta \cdot \frac{\pi}{180}} \\
   \theta &= shift\_degree 
   \end{align*}
   
Otherwise, the ratio does not include a phase shift:

.. math::
   :nowrap:
   
   \begin{align*}
   \underline{n} &= n
   \end{align*}
   
Impedance Values
------------------

The short-circuit impedance is calculated as:

.. math::
   :nowrap:

   \begin{align*}
   z_k &= \frac{vsc\_percent}{100} \cdot \frac{1000}{sn\_kva} \\
   r_k &= \frac{vscr\_percent}{100} \cdot \frac{1000}{sn\_kva} \\
   x_k &= \sqrt{z^2 - r^2} \\
   \underline{z}_k &= r_k + j \cdot x_k
   \end{align*}    

The magnetising admittance is calculated as:

.. math::
   :nowrap:

   \begin{align*}
   y_m &= \frac{i0\_percent}{100} \\
   g_m &= \frac{pfe\_kw}{sn\_kva \cdot 1000} \cdot \frac{1000}{sn\_kva} \\
   b_m &= \sqrt{y_m^2 - g_m^2} \\
   \underline{y_m} &= g_m - j \cdot b_m
   \end{align*}    

The values calculated in that way are relative to the rated values of the transformer. To transform them into the per unit system, they have to be converted to the rated values of the network:

.. math::
   :nowrap:

   \begin{align*}
    Z_{N} &= \frac{V_{N}^2}{S_{N}} \\
    Z_{ref, trafo} &= \frac{vn\_lv\_kv^2 \cdot 1000}{sn\_kva} \\
    \underline{z} &= \underline{z}_k \cdot \frac{Z_{ref, trafo}}{Z_{N}} \\
    \underline{y} &= \underline{y}_m \cdot \frac{Z_{N}}{Z_{ref, trafo}} \\
    \end{align*}

Where the reference voltage :math:`V_{N}` is the nominal voltage at the low voltage side of the transformer and the rated apparent power :math:`S_{N}` is defined system wide in the net object (see :ref:`Unit Systems and Conventions<conventions>`). 

Tap Changer
---------------

**Longitudinal regulator**

A longitudinal regulator can be modeled by setting tp_phase_shifter to False and defining the tap changer voltage step with tp_st_percent.

The reference voltage is then multiplied with the tap factor:

.. math::
   :nowrap:
   
   \begin{align*}
    n_{tap} = 1 + (tp\_pos - tp\_mid) \cdot \frac{tp\_st\_percent}{100}
    \end{align*}
    
On which side the reference voltage is adapted depends on the :math:`tp\_side` variable:

.. tabularcolumns:: |p{0.2\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|
.. csv-table:: 
   :file: trafo_tap.csv
   :delim: ;
   :widths: 20, 15, 15

.. note::
    The variables tp_min and tp_max are not considered in the power flow. The user is responsible to ensure that tp_min < tp_pos < tp_max!

**Cross regulator**

In addition to tp_st_percent a value for tp_st_degree can be defined to model an angle shift for each tap, resulting in a cross
regulator that affects the magnitude as well as the angle of the transformer ratio.


**Ideal phase shifter**

If tp_phase_shifter is set to True, the tap changer is modeled as an ideal phase shifter, meaning that a constant
angle shift is added with each tap step:

.. math::
   :nowrap:
   
   \begin{align*}
   \underline{n} &= n \cdot e^{j \cdot (\theta + \theta_{tp}) \cdot \frac{\pi}{180}} \\
   \theta &= shift\_degree 
   \end{align*}
   
The angle shift can be directly defined in tp_st_degree, in which case:

.. math::
   :nowrap:
   
   \begin{align*}
   \theta_{tp} = tp\_st\_degree \cdot (tp\_pos - tp\_mid)
   \end{align*}

or it can be given as a constant voltage step in tp_st_percent, in which case the angle is calculated as:

.. math::
   :nowrap:
   
   \begin{align*}
   \theta_{tp} = 2 \cdot arcsin(\frac{1}{2} \cdot \frac{tp\_st\_percent}{100})  \cdot (tp\_pos - tp\_mid)
   \end{align*}

If both values are given for an ideal phase shift transformer, the power flow will raise an error.

.. seealso::

    `ENTSO-E - Phase Shift Transformers Modelling, Version 1.0.0, May 2015 <https://docstore.entsoe.eu/Documents/CIM_documents/Grid_Model_CIM/ENTSOE_CGMES_v2.4_28May2014_PSTmodelling.pdf>`_
    
    `J. Verboomen, D. Van Hertem, P. H. Schavemaker, W. L. Kling and R. Belmans, "Phase shifting transformers: principles and applications," 2005 International Conference on Future Power Systems, Amsterdam, 2005 <https://ieeexplore.ieee.org/document/1600575/>`_




Result Parameters
==========================
*net.res_trafo*

.. tabularcolumns:: |p{0.15\linewidth}|p{0.10\linewidth}|p{0.55\linewidth}|
.. csv-table:: 
   :file: trafo_res.csv
   :delim: ;
   :widths: 15, 10, 55


.. math::
   :nowrap:
   
   \begin{align*}
    p\_hv\_kw &= Re(\underline{v}_{hv} \cdot \underline{i}^*_{hv}) \\    
    q\_hv\_kvar &= Im(\underline{v}_{hv} \cdot \underline{i}^*_{hv}) \\
    p\_lv\_kw &= Re(\underline{v}_{lv} \cdot \underline{i}^*_{lv}) \\
    q\_lv\_kvar &= Im(\underline{v}_{lv} \cdot \underline{i}^*_{lv}) \\
	pl\_kw &= p\_hv\_kw + p\_lv\_kw \\
	ql\_kvar &= q\_hv\_kvar + q\_lv\_kvar \\
    i\_hv\_ka &= i_{hv} \\
    i\_lv\_ka &= i_{lv}
    \end{align*}
    
The definition of the transformer loading depends on the trafo_loading parameter of the power flow.

For trafo_loading="current", the loading is calculated as:

.. math::
   :nowrap:
   
   \begin{align*}  
    loading\_percent &= max(\frac{i_{hv} \cdot vn\_hv\_kv}{sn\_kva}, \frac{i_{lv} \cdot vn\_lv\_kv}{sn\_kva})  \cdot 100
   \end{align*}
    

For trafo_loading="power", the loading is defined as:
    
.. math::
   :nowrap:
   
   \begin{align*}  
    loading\_percent &= max( \frac{i_{hv} \cdot v_{hv}}{sn\_kva}, \frac{i_{lv} \cdot v_{lv}}{sn\_kva}) \cdot 100
    \end{align*} 
