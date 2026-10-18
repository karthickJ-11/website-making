# DTMB 5415 calm-water resistance in STAR-CCM+

Setup notes for a DTMB 5415 model at scale 1:51.1 (Lpp 2.779 m), Fn = 0.39, using an overset mesh, DFBI free sinkage and trim, and VOF.

- [`REPORT.md`](REPORT.md): **final setup report** (mesh matched to IITM-RANS3D, standard k-ε at y⁺ = 30, 0.001 s time step, 20 s run), with every number derived. Where it differs from the two HTML files below, the report holds the later decisions.
- [`DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html`](DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html): earlier setup guide following the ITTC procedures, with a finer STAR-only grid family. It covers mesh sizing, prism layers, domain, physics, time step, DFBI ramp, residual targets, resistance extrapolation and a grid-convergence plan.
- [`DTMB5415_STARCCM_replicate_IITM-RANS3D.html`](DTMB5415_STARCCM_replicate_IITM-RANS3D.html): mesh and setup that match the earlier IITM-RANS3D run (30 mm lateral / 10 mm vertical refined region). Values marked ASSUMED still need the RANS3D inputs.

Open the HTML files in a browser. Hull particulars marked ⚑ in the guide still need checking against the geometry file.
