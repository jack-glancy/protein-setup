# System setup protocol
#### Written by Jack Glancy
***
## PDB Protonation
***
1. Download pdb file from Protein data bank. Ensure pdb is correct bilogical assembly (monomer, dimer, etc.) as well as correct ligand. 
2. Submit pdb file to H++ web server. (http://biophysics.cs.vt.edu/uploadpdb.php) 
4. Select pH that experiment was run at.
5. Process structure.
6. Download the output.pdb file.
> Note: If your pdb is made up of multiple models, as is the case for some biological 
> assemblies of dimers/trimers, then the easiest way I've found to get H++ to work 
> on each protein unit is to submit the original pdb (which will process the first model), 
> then delete the first model and resubmit (that'll process the second one) 
> and so on and then combine the outputted pdbs through concatenation.
7. This should give you a fully/correctly protonated output.pdb for the protein without any ligands. To ensure this structure is in amber format run:
```
$ pdb4amber output.pdb > output_amb.pdb
```
***
## Ligand Parametrisation
***
9. Now extract the pdb data for one of each ligand from the original pdb into respective ligand1.pdb, ligand2.pdb etc. For example, if you have two ligands with
   residue names 'NOS' and 'SO4' you can copy and paste from the original.pdb or, if they appear only once in your system, this can be done with:
```
$ grep 'NOS' original.pdb > NOS.pdb
$ grep 'SO4' original.pdb > SO4.pdb
```
10. Ensure you have only one iteration of your ligand in each ligand.pdb. Examples can be found of these pdbs in this repo.
11. To ensure they are in amber format run:
```
$ pdb4amber ligand.pdb > ligand_amb.pdb
```
12. For organic ligands, one can reduce the ligand_amb.pdb structure with the command:
```
$ reduce ligand_amb.pdb > ligand_amb_h.pdb
```
> Note: For inorganic ligands, this reduction step can be skipped.
13. This pdb should be visually inspected to check that the ligand is appropriately protonated. Also if the organic ligand is not neutral, this should be changed manually.
    I use the Chemcraft software, but this can also be done through the molefacture extension in VMD.
14. From the fully reduced ligand pdb, one can run antechamber to start parametrising your ligand. This is started by making a mol2 file.
```
$ antechamber -i ligand_amb_h.pdb -fi pdb -o ligand.mol2 -fo mol2 -c bcc -s 2
```
or if your ligand is charged add the -nc <charge> flag. Here we are parametrising a +1 cationic ligand.
```
$ antechamber -i ligand_amb_h.pdb -fi pdb -o ligand.mol2 -fo mol2 -c bcc -s 2 -nc 1
```
15. Amber can then check the mol2 for missing parameters using parmchk and generate a frcmod file containing these missing params.
```
$ parmchk2 -i ligand.mol2 -f mol2 -o ligand.frcmod
```
16. Once you have the frcmod, run tleap to generate a lib and topology file for the ligand.
```
$ tleap
$ source oldff/leaprc.ff99SB
$ source leaprc.gaff
$ LIG = loadmol2 ligand.mol2
$ check LIG
$ loadamberparams ligand.frcmod
$ saveoff LIG ligand.lib
$ saveamberparm LIG ligand.prmtop ligand.inpcrd
$ quit
```
> Note: The resname LIG needs to match the resname of the ligand in the pdb, as this will be referred to in the lib file. 
17. Generate lib, topology and crd files for all the ligands in your system.
***
## Final PDB Setup
***
18. Extract all the coordinates of the ligands from the original pdb into a separate pdb. This may have been accomplished in Step 9, if you only have one iteration of each ligand in your system.
19. Make sure these pdbs are in amber format with pdb4amber.
20. Concatenate the protonated protein (output_amb.pdb) and ligand pdbs into full.pdb.
21. Run pdb4amber on this to generate full_amb.pdb.
22. Once you have this, run tleap which will add mising hydrogens to your ligands, solvate the whole system in a water box, add counterions to neutralise the system and generate a topology and crd file for the system.
```
$ tleap
$ source leaprc.protein.ff14SB
$ source leaprc.water.spce
$ source leaprc.gaff
$ loadamberparams ligand1.frcmod
$ loadoff ligand1.lib
$ loadamberparams ligand2.frcmod
$ loadoff ligand2.lib
$ enz = loadPdb "full_amb.pdb"
$ solvateOct enz SPCBOX 14.0
$ addIons enz Na+ 9
$ saveAmberParm enz solvated_enz.top solvated_enz.crd
$ savepdb enz solvated_enz.pdb
$ quit
```
> Note: The '14.0' at the end of the solvateOct command specifies that the edge of the water box is no closer than 14.0 A from the farthest protein residue. On the addIons command, the Na+ is interchangeable with Cl- if the overall system is net positive prior to solvation, and the number '9' denotes the number of ions to add.  
23. All being well, this should output all the files needed for simulation.  
