# System setup
***
1. Download pdb file from Protein data bank. Ensure pdb is correct bilogical assembly (monomer, dimer, etc.) as well as correct ligand. 
2. Submit pdb file to H++ web server. (http://biophysics.cs.vt.edu/uploadpdb.php) 
4. Select pH that experiment was run at.
5. Process structure.
6. Download the output.pdb file.
```
    Note: If your pdb is made up of multiple models, as is the case for some biological 
    assemblies of dimers/trimers, then the easiest way I've found to get H++ to work 
    on each protein unit is to submit the original pdb (which will process the first model), 
    then delete the first model and resubmit (that'll process the second one) 
    and so on and then combine the outputted pdbs through concatenation.
```
7. This should give you a fully/correctly protonated output.pdb for the protein without any ligands. To ensure this structure is in amber format run:
```
$ pdb4amber output.pdb > output_amb.pdb
```
9. Now extract the pdb data for one of each ligand from the original pdb into respective ligand1.pdb, ligand2.pdb etc. For example, if you have ligands with
   residue names 'NOS' and 'SO4', you can copy and paste from the original.pdb or this can be done with:
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
