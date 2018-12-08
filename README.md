# OPLGen - Oligopeptide Ligand Generator

Simple command line tools to generate cyclic and linear oligopeptides in SMILES format, filter them based on drug-likeness, output stats and perform ligand file preparation using Open Babel to output ready-to-dock mol2 files. Supports Windows, OSX and Linux

Developed by:  
[@matthewjumpsoffbuildings](https://github.com/matthewjumpsoffbuildings)  
Ezra Rex  
Flinders University of South Australia  
College of Science and Engineering  
Molecular Biology and Biotechnology Research Cluster

In collaboration with  
Chinese Academy of Sciences  
Yantai Institute of Coastal Zone Research  
Department of Molecular Phycology

## Installation

Before installing, ensure you have the following requirements

- [node+npm](https://nodejs.org/en/download/) (recent versions)
- [openbabel](http://openbabel.org/wiki/Category:Installation) (for mol2 conversion)

Once you have the requirements, run the following in a cmd/terminal:

```
npm install -g oplgen
```

If you are on linux or OSX you may need to add `sudo` at the start of the install command like so:

```
sudo npm install -g oplgen
```

## Updating

To update to the latest version, use the following command (with `sudo` at the start if needed):

```
npm install -g oplgen@latest
```

## Usage

This program contains 3 command line utilities:

- [`opl-generate`](#opl-generate)   (alias [`oplgen`](#opl-generate))
- [`opl-filter`](#opl-filter)       (alias [`oplflt`](#opl-filter))
- [`opl-subunits`](#opl-subunits)   (alias [`oplsub`](#opl-subunits))

The correct procedure to use these commands is to create separate folders for each type/length of chain you are interested in generating/filtering. For example if you want to generate cyclic chains of length 5, make a folder named `cyclo.5` or something similar, and open a cmd/terminal in that folder to run the generation/filtering. Then if you want to generate linear chains of length 4 with `ADDA` conserved at position 1, make another folder named `linear.4.1:ADDA` and open a cmd/terminal there to do the generation/filtering for that chain type.

## opl-generate

Generate a specified number of oligopeptides in SMILES format from a collection of subunits stored in JSON format. Duplicates will not be created. For an example of how the subunits JSON is structured see the default JSON [here](./subunits.json)

Once you have run `oplgen` once in a working directory with some arguments, they will be stored in a `.params` file so if you want to generate more using the same arguments you can just run `oplgen` without having to respecify the settings. This is useful since the number of possible oligopeptides is huge for anything but very small lengths, so you may want to generate in batches. You can still specify a different value for `-n`/`--number` if you want to generate more/less each time, but you shouldnt change any of the other options in the current working directory. If you accidentally generated the wrong type of chain the first time you ran `oplgen` in a new folder, just delete the `smiles` folder and `.params` file and start again.

Once you have generated the desired number of SMILES, use [`opl-filter`](#opl-filter) to select a subset of them based on drug-likeness, output stats about them, and convert them to mol2 using openbabel

The following options are available:

- `-n --number` - default: `100000`
	- How many SMILES to generate? Given even relatively short chain lengths, the number of possible unique chains is often beyond a home computers capability to generate, so this defaults to `100000`
	- If you pass a number greater than the possible number of unique chains for the given length/subunits, it will generate all possible unique chains
	- If you pass a number less than 1, it will generate all possible unique chains for the given length/subunits
- `-l --sequenceLength` - default: `5`
	- How long should each generated chain of subunits be?
- `-s --subunits` - default: [`subunits.json`](./subunits.json)
	- Where should the generator look for the subunits JSON data?
	- By default, the program has a subunits JSON file with 32 common subunits in its install directory
	- If no `subunits.json` file is present, or if you dont pass anything for this argument, the [default subunits JSON](./subunits.json) is used
	- If you pass a reference to a different JSON for this argument, it will attempt to find the file you specified, else it will fall back to using the default JSON
	- If you want to customise the default subunits JSON, you can use the [`opl-subunits`](#opl-subunits) command. This will insert the default JSON into the current working directory, for you to add/remove/modify subunits as needed
- `-c --conserve`
	- By default all chains are generated using randomly selected subunits from the subunits JSON.
	- If you want to ensure 1 or more specific subunits is present in all chains generated, you can use this argument
	- This argument takes the form `-c POSITION:SUBUNIT,POSITION:SUBUNIT`
	- For example to conserve `ADDA` at position 1 of all generated chains: `-c 1:ADDA`
	- To conserve `ADDA` at position 2 and `D-ALA` at position 5: `-c 2:ADDA,5:D-ALA`
- `--linear`
	- By default the chains generated will be cyclic. Passing the `--linear` argument makes the generated chains linear
- `-r --ringClosureDigit` - default: `9`
	- Unlikely to require editing.
	- The digit used to close cyclic chains in the generated SMILES files

#### Some Examples

Generate SMILES for 100,000 cyclic oligopeptides of length 5 (default):

```
opl-generate
```

Generate SMILES for 100,000 linear oligopeptides of length 7:

```
opl-generate -l 7 --linear
```

Generate SMILES for 20,000 cyclic oligopeptides of length 6 with the ADDA subunit conserved at position 1 and D-ALA at position 4:

```
opl-generate -n 20000 -c 1:ADDA,4:D-ALA
```

## opl-filter

Take a large number of SMILES files generated by [`opl-generate`](#opl-generate), sort them by drug-likeness, select a range from the top scorers, output stats about this selected range, and optionally convert them to mol2 and create an `output.mol2` ligand file ready for dock6. If an `output.mol2` file already exists in the current working directory, newly converted mol2s will be appended to it. As with [`opl-generate`](#opl-generate) duplicates are not created, and you may want to do this stage in batches since the openbabel mol2 conversion may take while for each SMILES file. You can also use a tool like [UCSF Chimera](https://www.cgl.ucsf.edu/chimera/download.html) to view individual mol2 files that are output in the `mol2` folder

This command generates 2 stats files, `stats-totals.txt` and `stats-files.txt`, both containing data about the values of the properties of the SMILES that `opl-filter` picked. Both will contain a header line with a description of, and the command used to create that data set, followed by CSV style data about the selected SMILES. If you are interested in the specific property values of each SMILES file that was chosen, look in `stats-files.txt`. If you want to see statistics such as mean, median, min, max and standard deviations for _all_ the SMILES chosen by this filter run, look in `stats-totals.txt`. If you run `opl-filter` multiple times, the new stats will be appended at the end of the stats txt files.

The following options are available:

- `-n --number` - default: `100`
	- How many of the generated SMILES do we want to convert to mol2? Usually this will be a small sub-set of the total number of generated SMILES.
	- If you want to process every SMILES file, pass `-n 0`
- `-r --range` - default: `same as --number`
	- After the program sorts all the SMILES on drug-likeness, what range of the sorted SMILES should the final 100 (or whatever amount we requested with the `-n`/`--number` argument) be selected from?
	- By default the range is set to the same as the requested `-n`/`--number`, meaning you get the exact top 100 of the sorted list
	- Since this means that there will be a lot less diversity, you may wish to make the range larger than the desired number of mol2 files, but smaller than the total amount of generated SMILES
	- For example, if you have generated 1,000,000 SMILES, and you want to sort them and turn 100 of them into mol2 files for docking, or generate stats, and you want a bit more diversity, you could set the range to 1000. This will select 100 SMILES randomly from the top 1000 SMILES files, as opposed to picking the exact top 100.
	- As with `--number`, if you pass `-r 0` this will set the range to include all SMILES files.
- `-s --stats`
	- By default `oplflt` will sort, generate stats, and convert to mol2.
	- If you just want to do a sort and generate stats, but dont want to convert to mol2, pass `-s` or `--stats`

#### Some Examples:

Generate stats and mol2 files for the exact top scoring 100 SMILES files (default):

```
opl-filter
```

Generate stats and mol2 files for 200 SMILES files randomly selected from the top scoring 1000:

```
opl-filter -n 200 -r 1000
```

Generate only stats for 300 SMILES randomly selected from all available SMILES files:

```
opl-filter -n 300 -r 0 -s
```

Generate only stats for all available SMILES files:

```
opl-filter -n 0 -s
```

## opl-subunits

Copy the [default subunits.json](./subunits.json) into the current working directory. You can then edit it as needed, and [`opl-generate`](#opl-generate) and [`opl-filter`](#opl-filter) will use your local copy when running in that directory, instead of the default subunits JSON file

```
opl-subunits

oplsub
```
