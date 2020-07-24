# Protocol capture
## Introduction

The following is a protocol capture of the analysis of VHH structures from the publication *Structure- and sequence-based design of synthetic single-domain antibody libraries*, from Sevy et al., PEDS 2020. It will review the Rosetta protocols used for VHH analysis. All Rosetta commands for this publication were run with version 3.8, from December 2017. All materials from this protocol capture can be downloaded from
[https://github.com/sevya/VHH_modeling]().


## Structure download and preparation
The structures used in this study were downloaded from the [Structural Antibody Database (SabDab)](http://opig.stats.ox.ac.uk/webapps/newsabdab/sabdab/). Search results were filtered by the following criteria: VHH antibodies; sub-3.5 Å resolution; and antigen type protein or peptide. Results were filtered manually by VHH sequence to produce a set of nonredundant structures. The structures were downloaded from the [Protein DataBank](http://www.rcsb.org) and manually processed to remove water and non-protein residues and renumbered starting from residue 1.

Structures were prepared using a constrained FastRelax refinement protocol, using constraints to starting coordinates of backbone heavy atoms, with a standard deviation of 0.5 Å. 10 relaxed structures were created for each input complex, and the best scoring decoy was used for binding energy analysis. Shown below are the XML script and command to run relaxation. 

relax.command:

`rosetta_scripts.default.linuxgccrelease @relax.options -s 1bzq_HA.pdb -parser:protocol relax_cst.xml`

relax.options:

`-in:file:fullatom
-out:file:fullatom
-out:pdb_gz
-ex1
-use_input_sc
-nstruct 10
-suffix _relax_iface
-scorefile relax_iface.fasc
-out:path:pdb models/
`

relax_cst.xml:

	<ROSETTASCRIPTS>
		<SCOREFXNS>
			<ScoreFunction name="ref_cst" weights="ref2015_cst.wts" />
			<ScoreFunction name="ref" weights="ref2015.wts" />
		</SCOREFXNS>
		<FILTERS>
		</FILTERS>
		<TASKOPERATIONS>
			<InitializeFromCommandline name="ifcl"/>
			<RestrictToRepacking name="rtr"/>
			<RestrictToInterfaceVector name="rtiv" chain1_num="1" chain2_num="2" CB_dist_cutoff="10.0" nearby_atom_cutoff="5.5" vector_angle_cutoff="75" vector_dist_cutoff="9.0" />
		</TASKOPERATIONS>
		<MOVERS>
			<PackRotamersMover name="repack" task_operations="ifcl,rtr" scorefxn="ref" />
			<FastRelax name="relax" task_operations="ifcl,rtr,rtiv" scorefxn="ref_cst" >
				<MoveMap name="mm" >
					<Jump number="1" setting="0" />
				</MoveMap>
			</FastRelax>
			<AtomCoordinateCstMover name="cst" />
			<VirtualRoot name="root" removable="1" />
			<VirtualRoot name="rmroot" remove="1" />
		</MOVERS>
		<PROTOCOLS>
			<Add mover_name="repack" />
			<Add mover_name="root" />
			<Add mover_name="cst" />
			<Add mover_name="relax"/>
			<Add mover_name="rmroot" />
		</PROTOCOLS>
		<OUTPUT scorefxn="ref" />
	</ROSETTASCRIPTS>


## Binding energy measurement
To measure binding energy along the VHH-antigen interface, the following XML script and commands were used. This script will measure the binding energy across the interface and break it down to a per-residue basis.

measure_ddg.command:

`rosetta_scripts.mpi.linuxgccrelease @measure_ddg.options -l best_decoys.list -scorefile ddg.fasc -parser:protocol measure_ddg.xml`

measure_ddg.options:

`-in:file:fullatom
-out:file:fullatom
-out:pdb_gz
-ex1
-use_input_sc
-nstruct 1
-out:file:score_only
-out:no_nstruct_label
`

measure_ddg.xml:

	<ROSETTASCRIPTS>
		<SCOREFXNS>
			<ScoreFunction name="ref_cst" weights="ref2015_cst.wts" />
			<ScoreFunction name="ref" weights="ref2015.wts" />
		</SCOREFXNS>
		<FILTERS>
		</FILTERS>
		<TASKOPERATIONS>
			<InitializeFromCommandline name="ifcl"/>
			<RestrictToRepacking name="rtr"/>
			<RestrictToInterfaceVector name="rtiv" chain1_num="1" chain2_num="2" CB_dist_cutoff="10.0" nearby_atom_cutoff="5.5" vector_angle_cutoff="75" vector_dist_cutoff="9.0" />
		</TASKOPERATIONS>
		<MOVERS>
			<ddG name="ddg" per_residue_ddg="1" repack_unbound="1" chain_num="1" task_operations="rtiv,ifcl,rtr" scorefxn="ref" />
		</MOVERS>
		<PROTOCOLS>
			<Add mover_name="ddg" />
		</PROTOCOLS>
		<OUTPUT scorefxn="ref" />
	</ROSETTASCRIPTS>