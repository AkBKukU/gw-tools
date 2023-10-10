# gw-tools
This is a collection of tools for automating the ripping of floppy disks with
built in error detection and validation depending on the format of the disk.
This is made possible with the [Greaseweazle](https://github.com/keirf/greaseweazle)
project.

### Why?
I work with a wide variety of disk types and prefer to validate my disk image
archives by using tools to "use" the files somehow. Most of the time that means
extracting files from the disk image.

I also sometimes need to create my own `diskdef`s for disks for uncommon
systems.

This is a location for me to track the different methods I'm using for ripping
and validating disks based on the different systems and formats.

## Project Structure
There are two types of scripts in this repo, the single **parent** script, and
multiple **format** scripts. 

### Parent Script
The parent script is `gw-raw`. This script handles the primary reading of a disk
to get the best archival copy to start. It is used by all format scripts as 
well. If an error is detected while decoding a format it will automatically 
re-rip the disk with more revs to get a higher confidence copy to work with. Any
tasks associated with all disk imaging for every format goes here.

The parent script can also be directly called to just get a raw reading of a
disk if you do not know the format or do not wish to decode it.

The parent script should not normally need to be modified.

### Format Scripts
All other scripts will `source` the parent script and call its `rip` function.
They can also set up there own variables for file names. Log file names and
other information can be sourced from the parent script.

#### Function Overloading
Format scripts need to overload the `decode` function from the parent script.
This will most likely just be a call using `gw` with a diskdef for the specific
format. You also can call any other external utilities needed to get your final 
usable image. The decode log should always contain a `gw convert` output which
is used to validate the image.

Format scripts may also overload the `extract` function to perform post
processing work to images. This is done after validation and potential
re-rips. Logging here is optional.

#### Custom Diskdefs
If a format does not have a matching `gw` diskdef one can be added to the 
`diskdefs.cfg` file. The `gw-raw` script provides a path to the file in a 
`$diskdefs` variable by determining where it was run from.

## Usage
Just running `gw-ibm.1440 disk-name` as an example is all you need to do 
depending on the format. This would create Kryoflux raw tracks, a 1.44MB IMG, 
and extract files using `mtools`.

The format scripts should have a header for sourcing the parent file that will
work either in the local directory or when the `gw-tools` folder is put in the
`PATH`.

If you are adding a new format script, you can follow these steps:

1. Rip your unknown disk using `gw-raw $project_name`
2. Inspect the disk structure using something like [HxC Floppy Emulator](https://github.com/jfdelnero/HxCFloppyEmulator)
3. Create a new format script (copy `gw-template` as a template)
4. Run your new format script using the same project name to have it finish the decoding process.

## Currently Supported Formats

| Disk Type               | Format Script               |
|-------------------------|-----------------------------|
|IBM 3.5in DSHD 1.44MB    | `gw-ibm.1440`               |
|Jonos Ltd 3.5in SSDD     | `gw-jonos.35`               |
|HP LIF 33 Track DD       | `gw-hp.lif.33dd`            |
|HP LIF 77 Track DD       | `gw-hp.lif.77dd`            |

