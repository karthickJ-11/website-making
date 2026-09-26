# DTMB 5415 calm-water resistance in STAR-CCM+

Setup notes for a DTMB 5415 model at scale 1:51.1 (Lpp 2.779 m), Fn = 0.39, using an overset mesh, DFBI free sinkage and trim, and VOF.

- [`DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html`](DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html): full setup guide following the ITTC procedures. It covers mesh sizing, prism layers, domain, physics, time step, DFBI ramp, residual targets, resistance extrapolation and a grid-convergence plan.
- [`DTMB5415_STARCCM_replicate_IITM-RANS3D.html`](DTMB5415_STARCCM_replicate_IITM-RANS3D.html): mesh and setup that match the earlier IITM-RANS3D run (30 mm lateral / 10 mm vertical refined region). Values marked ASSUMED still need the RANS3D inputs.

Open either file in a browser. Hull particulars marked ⚑ in the guide still need checking against the geometry file.
