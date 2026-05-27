---
layout: post-en
title:  "Semiconductor Physics: From Silicon To Diodes"
date:   2026-05-25 18:00:00 +0800
categories: en-us
tags: electronics semiconductor notes
math: True
---

This is a summary note of the semiconductor physics part in the course "Ve311 Electronic Circuits."

## Quantum View of Silicon

### Valence & Conduction Bands

Silicon has an atomic number of 14, and its electron configuration is given by $$[\text{Ne}]3\text{s}^2 3\text{p}^2$$. In an isolated silicon atom, the electrons' wavefunctions and energy levels are cleanly quantized and discrete, so there are no special effects worth attention.

However, in a silicon crystal where the interatomic distance is comparable to the size of atoms, the wavefunctions of different silicon atoms overlap significantly. This will lead to merging and recombination, and the net result is

- a **valence band** at the *lower* energies, and
- a **conduction band** at the *higher* energies.

A *band* is an energy region containing many possible energy levels. 

Between the valence band and the conduction band, there is a **bandgap**, in which there are no energy levels existing. The size of the bandgap is measured by the **energy difference** between the two bands, typically denoted as $$E_g$$. For silicon, $$E_g = 1.12\,e\mathrm{V}$$ at room temperature.

<p style="text-align: center;"><img src="/assets/images/silicon-diode/bandgap.png" width="400"></p>
<p style="text-align: center; color: gray;"><small>Visualization of two bands and bandgap. (Picture taken from lecture slides)</small></p>

### Electrons and Holes

At $$0\,\text{K}$$, the valence band is completely filled, and the conduction band is completely empty. 

As temperature gradually increases, some **electrons** will be excited and enter the conduction band, leaving behind **holes** in the valence band. The negatively charged electrons can move freely in the conduction band, and the positively charged holes can also "move" freely in the valence band, so they both act as current carriers.

> 💡 Actually, in the conduction band, it is still the electrons that are actually moving, occupying holes and leaving behind new ones. However, that is difficult to describe quantitatively. Therefore, the equivalent description using holes is used.



